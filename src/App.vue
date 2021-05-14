<template>
  <v-app>
    <v-app-bar
        app
        color="primary"
        dark
        flat
    >
      <div class="d-flex align-center">
        <h2>Crawl Data</h2>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <Crawler/>
    </v-main>
  </v-app>
</template>

<script>
import Crawler from './components/Crawler';

export default {
  name: 'PexelsCrawler',

  components: {
    Crawler,
  },

  data: () => ({
    //
  }),
};
</script>
