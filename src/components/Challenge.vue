<template>
  <!-- Desafio -->
  <v-row
    justify="center"
    class="grey lighten-4"
    style="width:70%; font-size: 1.2rem; position: relative; user-select: none; "
  >
    <v-col cols="12" class="white"><b>Desafio</b></v-col>
    <v-col
      v-for="linha1 in selectedChallenge[0]"
      :key="linha1"
      cols="4"
      :class="linha1 == '' ? 'red lighten-4 elevation-1' : 'elevation-1'"
      >{{ linha1 }}
    </v-col>
    <v-col
      v-for="linha2 in selectedChallenge[1]"
      :key="linha2"
      cols="4"
      :class="linha2 == '' ? 'red lighten-4 elevation-2' : 'elevation-1'"
      >{{ linha2 }}
    </v-col>
    <v-col
      v-for="linha3 in selectedChallenge[2]"
      :key="linha3"
      cols="4"
      :class="linha3 == '' ? 'red lighten-4 elevation-1' : 'elevation-1'"
      >{{ linha3 }}
    </v-col>
    <v-tooltip color="grey darken-2" bottom nudge-bottom="-8">
      <template v-slot:activator="{ on }">
        <v-icon
          v-on="on"
          color="grey darken-2"
          style="position: absolute; top: 7%; right: 0"
          @click="customPuzzle"
          >mdi-pencil</v-icon
        >
      </template>
      <span>Puzzle Personalizado</span>
    </v-tooltip>
  </v-row>
  <!-- Fim Desafio -->
</template>

<script>
export default {
  name: 'Challenge',
  props: {
    selectedChallenge: {
      type: Array,
      default: () => {
        return [];
      },
    },
  },
  methods: {
    customPuzzle() {
      this.$emit('customPuzzle');
    },
  },
};
</script>

<style></style>
