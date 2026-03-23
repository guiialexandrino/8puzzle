<template>
  <v-dialog v-model="dialogModel" retain-focus persistent width="500px">
    <v-card class="pa-6" align="center">
      <v-icon style="transform: scale(1.3);" class="mb-3" x-large color="yellow darken-2">
        mdi-alert
      </v-icon>
      <p>
        <span>
          <b>Ops...</b> o algoritmo não conseguiu encontrar a solução :(
          <br />
        </span>
      </p>
      <v-btn
        rounded
        class="mt-4 white--text"
        elevation="0"
        color="yellow darken-2"
        @click="dialogModel = false"
      >
        Ok
      </v-btn>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: 'AlertDialog',
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    dialogModel: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    }
  }
}
</script>
