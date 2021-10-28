<template>
  <v-app>
    <v-main>
      <v-container fill-height>
        <EightPuzzle />
      </v-container>
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

<style>
* {
  user-select: none;
}
</style>
