<template>
  <v-dialog v-model="dialogModel" retain-focus persistent width="500px">
    <v-card class="pa-6" align="center">
      <v-icon style="transform: scale(1.3);" class="mb-3" x-large color="green lighten-2">
        mdi-puzzle-edit
      </v-icon>
      <br />
      <span style="font-size: 1.2rem"><b>Desafio Personalizado</b></span>
      <br />
      <br />
      <p>
        <span
          >Para representar a casa sem numero digite 0 no lugar.<br />Um <b>exemplo valido</b> de
          desafio e "123405786" <br
        /></span>
      </p>
      <v-text-field
        style="max-width: 70%"
        :counter="9"
        :error="false"
        :error-messages="errorMessage"
        maxlength="9"
        v-model="newCustomPuzzle"
        @keypress="validation($event)"
      ></v-text-field>
      <v-btn
        :disabled="newCustomPuzzle.length < 9"
        rounded
        class="mt-4 mr-2 white--text"
        elevation="0"
        color="green lighten-2"
        @click="validationCustomPuzzle"
        >Pronto</v-btn
      >
      <v-btn rounded class="mt-4" elevation="0" outlined @click="handleCancel">Cancelar</v-btn>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: 'CustomPuzzleDialog',
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data: () => ({
    newCustomPuzzle: '',
    errorMessage: ''
  }),
  computed: {
    dialogModel: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      }
    }
  },
  methods: {
    validation($event) {
      const keyCode = $event.keyCode ? $event.keyCode : $event.which
      if (keyCode < 48 || keyCode > 56) {
        $event.preventDefault()
      }
    },

    validationCustomPuzzle() {
      const set = new Set()
      let check = 0
      const numbers = this.newCustomPuzzle.toString()
      for (let i = 0; i < numbers.length; i++) {
        if (!set.has(numbers[i])) {
          check++
          set.add(numbers[i])
        }
      }

      if (check == 9 && numbers !== '123456780') {
        this.errorMessage = ''
        const puzzle = [
          [
            numbers[0] !== '0' ? numbers[0] : '',
            numbers[1] !== '0' ? numbers[1] : '',
            numbers[2] !== '0' ? numbers[2] : ''
          ],
          [
            numbers[3] !== '0' ? numbers[3] : '',
            numbers[4] !== '0' ? numbers[4] : '',
            numbers[5] !== '0' ? numbers[5] : ''
          ],
          [
            numbers[6] !== '0' ? numbers[6] : '',
            numbers[7] !== '0' ? numbers[7] : '',
            numbers[8] !== '0' ? numbers[8] : ''
          ]
        ]

        this.$emit('custom-puzzle-created', {
          label: `Personalizado - ${numbers}`,
          puzzle
        })

        this.resetForm()
        this.dialogModel = false
      } else {
        this.errorMessage = 'Puzzle personalizado invalido.'
      }
    },

    handleCancel() {
      this.resetForm()
      this.dialogModel = false
    },

    resetForm() {
      this.newCustomPuzzle = ''
      this.errorMessage = ''
    }
  }
}
</script>
