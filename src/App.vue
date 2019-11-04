<template>
  <v-app id="app">
      <v-app-bar
        app
        dense
        class="elevation-0"
      >
        <v-toolbar-title>
          <strong>SIMBA</strong> Live
        </v-toolbar-title>
        <div class="flex-grow-1"></div>
        <v-btn text icon depressed href="https://github.com/SIMBAChain/SimbaLive" target="_blank">
          <v-icon>mdi-github-circle</v-icon>
        </v-btn>
        <v-btn text icon depressed :color=colorValue href="https://app.simbachain.com/" target="_blank" @click="colorValue = 'pink'">
          <v-icon>mdi-heart</v-icon>
        </v-btn>
      </v-app-bar>
   <v-content>
      <v-container
        class="fill-height"
        fluid
      >
        <router-view>
        </router-view>
      </v-container>
    </v-content>

  </v-app>
</template>

<script>

export default {
  name: 'app',
  created () {
    this.$vuetify.theme.dark = true
  },
  props: {
    source: String
  },
  data: () => ({
    drawer: null,
    colorValue: ''
  })
}
</script>

<style scoped>



</style>