<template>
  <v-app>
    <v-main>
      <EightPuzzle />
    </v-main>
  </v-app>
</template>

<script>
import EightPuzzle from './components/8Puzzle';

export default {
  name: 'App',

  components: {
    EightPuzzle,
  },

  data: () => ({
    //
  }),
};
</script>
