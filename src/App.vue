<template>
  <v-app>
    <v-toolbar color="teal darken-2" dark fixed app>
      <img class="logo" src="./assets/vue-graphql.png" alt="Logo">
      <v-toolbar-title class="hidden-xs-only">VueJS Roadtrip Heroes</v-toolbar-title>
    </v-toolbar>
    <v-content class="teal lighten-3">
      <v-container fluid class="app-container white" fill-height grid-list-md>
        <v-layout class="hero-cards-layout" wrap v-if="allHeroes.length">
          <template v-for="hero in allHeroes">
            <vue-hero :hero="hero" @deleteHero="deleteHero($event)" :key="hero.name"></vue-hero>
          </template>
        </v-layout>
        <v-dialog v-model="dialog" width="800" v-if="allHeroes.length">
          <v-btn slot="activator" color="teal" dark>Add Hero</v-btn>
          <v-card class="dialog-form">
            <v-form v-model="valid">
              <v-text-field v-model="name" :rules="nameRules" label="Name" required></v-text-field>
              <v-text-field v-model="image" label="Image Link"></v-text-field>
              <v-text-field v-model="twitter" label="Twitter"></v-text-field>
              <v-text-field v-model="github" label="Github"></v-text-field>
            </v-form>
            <v-card-actions>
              <v-btn :disabled="!valid" @click="addHero">submit</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <h3 class="headline" v-else>No heroes found ☹️</h3>
      </v-container>
    </v-content>
    <v-footer color="teal darken-2" app>
      <span class="footer-text text-xs-center white--text">&copy; 2018</span>
    </v-footer>
  </v-app>
</template>

<script>
import VueHero from "./components/VueHero";
import gql from "graphql-tag";

const fragment = gql`
  fragment Hero on VueHero {
    id
    name
    twitter
    github
    image
  }
`;

const ALL_HEROES_QUERY = gql`
  query {
    allHeroes {
      ...Hero
    }
  }
  ${fragment}
`;

const ADD_HERO_MUTATION = gql`
  mutation AddHero($hero: HeroInput!) {
    addHero(hero: $hero) {
      ...Hero
    }
  }
  ${fragment}
`;

const DELETE_HERO_MUTATION = gql`
  mutation DeleteHero($name: String!) {
    deleteHero(name: $name)
  }
`;

export default {
  name: "app",
  data() {
    return {
      valid: false,
      dialog: false,
      name: "",
      nameRules: [v => !!v || "Name is required"],
      image: "",
      github: "",
      twitter: "",
      allHeroes: []
    };
  },
  components: {
    VueHero
  },
  apollo: {
    allHeroes: {
      query: ALL_HEROES_QUERY
    }
  },
  methods: {
    addHero() {
      const hero = {
        name: this.name,
        image: this.image,
        twitter: this.twitter,
        github: this.github
      };
      this.$apollo.mutate({
        mutation: ADD_HERO_MUTATION,
        variables: {
          hero
        },
        update: (store, { data: { addHero } }) => {
          const data = store.readQuery({ query: ALL_HEROES_QUERY });
          data.allHeroes.push(addHero);
          store.writeQuery({ query: ALL_HEROES_QUERY, data });
        },
        optimisticResponse: {
          __typename: "Mutation",
          addHero: {
            __typename: "VueHero",
            id: -1,
            ...hero
          }
        }
      });
      this.dialog = false;
      this.name = "";
      this.image = "";
      this.github = "";
      this.twitter = "";
    },
    deleteHero(name) {
      this.$apollo.mutate({
        mutation: DELETE_HERO_MUTATION,
        variables: {
          name
        },
        update: store => {
          const data = store.readQuery({ query: ALL_HEROES_QUERY });
          const heroToDelete = data.allHeroes.find(hero => hero.name === name);
          data.allHeroes.splice(data.allHeroes.indexOf(heroToDelete), 1);
          store.writeQuery({ query: ALL_HEROES_QUERY, data });
        }
      });
    }
  }
};
</script>

<style lang="scss">
.container.app-container.fluid.fill-height {
  align-items: start;
  max-width: 1240px;
  padding: 30px;
  flex-direction: column;
}

.footer-text {
  padding-left: 20px;
}

.logo {
  display: block;
  max-height: 100%;
  height: 50px;
  box-sizing: border-box;
  border: 1px solid transparent;
}

.dialog-form {
  padding: 15px;
}

.hero-cards-layout {
  width: 100%;
  .v-card__title.hero-title {
    padding-bottom: 0;
  }

  .hero-icons {
    font-size: 24px;
    padding-top: 10px;
    a {
      color: inherit;
      padding-right: 10px;
      &:hover {
        color: teal;
      }
      &:focus,
      &:active {
        color: inherit;
      }
    }
  }
}

@media (max-width: 480px) {
  .logo {
    height: 36px;
    margin-right: 20px;
  }
}
</style>
