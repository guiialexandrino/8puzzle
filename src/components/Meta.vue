<template>
  <!-- Mostra Meta -->
  <v-row
    justify="center"
    class="grey lighten-4"
    style="width:70%; font-size: 1.2rem"
  >
    <v-col cols="12" class="white"><b>Objetivo</b></v-col>
    <v-col cols="4">{{ meta[0][0] }}</v-col>
    <v-col cols="4">{{ meta[0][1] }}</v-col>
    <v-col cols="4">{{ meta[0][2] }}</v-col>
    <v-col cols="4">{{ meta[1][0] }}</v-col>
    <v-col cols="4">{{ meta[1][1] }}</v-col>
    <v-col cols="4">{{ meta[1][2] }}</v-col>
    <v-col cols="4">{{ meta[2][0] }}</v-col>
    <v-col cols="4">{{ meta[2][1] }}</v-col>
    <v-col cols="4" class="blue lighten-4">{{ meta[2][2] }}</v-col>
  </v-row>
  <!-- Fim Meta -->
</template>

<script>
export default {
  name: 'Meta',
  props: {
    meta: {
      type: Array,
      default: () => {
        return [];
      },
    },
  },
};
</script>

<style></style>
