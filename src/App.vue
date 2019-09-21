<template>
  <v-app>
    <v-app-bar app>
      <v-spacer />
      <v-toolbar-title class="headline text-uppercase">
        <v-container align-center>
          <span>Bitcoin Rocket Game</span>
        </v-container>
      </v-toolbar-title>
      <v-spacer />
    </v-app-bar>

    <v-content>
      <Game />
    </v-content>
  </v-app>
</template>

<script>
import Game from "./components/Game";

export default {
  name: "App",
  components: {
    Game
  },
  data: () => ({
    //
  }),
  created() {
    this.$vuetify.theme.dark = true;
  }
};
</script>
