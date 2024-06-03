<template>
  <v-app style="background-color: #ffffff" class="ai-font">
    <v-app-bar
    app
      color="white"
      class="app-content-padding"
      hide-on-scroll
      elevate-on-scroll
      flat
    >
      <v-toolbar-title class="" style="font-weight: 500;">GenAI Product Finder</v-toolbar-title>
      <v-spacer></v-spacer>
      <p class="text-right mb-0 developer-font">
        Developed By
        <a
          href="https://vrijraj.xyz"
          target="_blank"
          style="color: #f9482f; text-decoration: none"
          >Vrijraj</a
        >
      </p>
    </v-app-bar>
    <v-main>
      <router-view class="app-content-padding" />
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    //
  }),
};
</script>

<style>
.v-toolbar .v-toolbar__content {
  padding-left: 0%;
  padding-right: 0%;
}
</style>
