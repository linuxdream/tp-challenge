<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <span class="mr-2">Trevor's Ice Cream Maker Simulator</span>
        <v-icon>mdi-storefront</v-icon>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <IceCream />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import IceCream from "./components/IceCream.vue";

export default Vue.extend({
  name: "App",

  components: {
    IceCream,
  },

  data: () => ({
    //
  }),
});
</script>
