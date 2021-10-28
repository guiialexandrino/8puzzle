<template>
  <!-- Desafio -->
  <v-row style="position: relative; user-select: none;" class="green lighten-5">
    <v-col cols="12" class="white"
      ><b>Resolução passo-passo ({{ finalPath.length - 1 }})</b></v-col
    >
    <v-col
      v-for="linha1 in resolution.puzzle[0]"
      :key="linha1"
      cols="4"
      :class="linha1 == '' ? 'green lighten-3 elevation-2' : 'elevation-1'"
      >{{ linha1 }}
    </v-col>
    <v-col
      v-for="linha2 in resolution.puzzle[1]"
      :key="linha2"
      cols="4"
      :class="linha2 == '' ? 'green lighten-3 elevation-2' : 'elevation-1'"
      >{{ linha2 }}
    </v-col>
    <v-col
      v-for="linha3 in resolution.puzzle[2]"
      :key="linha3"
      cols="4"
      :class="linha3 == '' ? 'green lighten-3 elevation-2' : 'elevation-1'"
      >{{ linha3 }}
    </v-col>

    <v-icon
      :disabled="resolution.id.length == 1"
      color="black"
      style="position: absolute; left: -16%; bottom: 25%"
      @click="clickPrevious"
      >mdi-arrow-left
    </v-icon>
    <v-icon
      :disabled="resolution.puzzle.toString() === meta.toString()"
      color="black"
      style="position: absolute; right: -16%; bottom: 25%"
      @click="clickNext"
      >mdi-arrow-right
    </v-icon>
  </v-row>
  <!-- Fim Desafio -->
</template>

<script>
export default {
  name: 'Resolution',
  props: {
    resolution: {
      type: Object,
      default: () => {
        return {};
      },
    },
    finalPath: {
      type: Array,
      default: () => {
        return [];
      },
    },
    meta: {
      type: Array,
      default: () => {
        return [];
      },
    },
  },
  methods: {
    clickNext() {
      this.$emit('nextStep');
    },
    clickPrevious() {
      this.$emit('previousStep');
    },
  },
};
</script>

<style></style>
