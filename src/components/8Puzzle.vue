<template>
  <v-container>
    <v-card class="py-8 px-10" elevation="2">
      <v-row>
        <v-col class="my-1" cols="12" align="center">
          <h1 class="display-2 font-weight-bold mb-3">
            8-PUZZLE
          </h1>
          <p class="subheading font-weight-regular">
            Trabalho disciplina <b>INE 5633</b> - Sistemas inteligentes.<br />
            <span style="font-size:0.9rem">Desenvolvido por: <b>Guilherme Alexandrino</b></span>
          </p>
          <v-row justify="center" no-gutters>
            <v-col cols="6" sm="6" md="4" lg="4" xl="3">
              <v-select
                :disabled="start"
                class="px-4"
                v-model="type"
                :items="typeOptions"
                item-text="label"
                item-value="type"
              />

              <Meta :meta="meta" />
            </v-col>
            <v-col cols="6" sm="6" md="4" lg="4" xl="3">
              <v-select
                :disabled="start"
                class="px-4"
                v-model="selectedChallenge"
                :items="challengeByAlgorithm"
                item-text="label"
                item-value="puzzle"
              />
              <Challenge :selectedChallenge="selectedChallenge" @customPuzzle="customPuzzle" />
            </v-col>
          </v-row>
        </v-col>
        <v-col
          cols="12"
          class="my-4"
          justify="center"
          style="display:flex; justify-content: center"
        >
          <v-btn
            :disabled="done"
            rounded
            elevation="0"
            color="blue lighten-1"
            class="white--text"
            @click="handleStart"
            >Resolver</v-btn
          >
          <v-btn
            rounded
            elevation="0"
            color="red lighten-2"
            class="white--text ml-3"
            @click="handleReset"
            >Resetar</v-btn
          >
        </v-col>
      </v-row>
      <v-row no-gutters justify="center" style="font-size:0.85rem">
        <span v-if="type == 'uniformCost' && !done">
          <b>**Atenção**</b> desafios em <u>Custo Uniforme</u> com
          <b>mais de 11 movimentos</b> tendem a demorar, é necessário aumentar o nª máximo de
          iterações.
        </span>
        <span v-if="type == 'a*' && !done">
          <b>**Atenção**</b> desafios em <u>A* Heuristica Simples</u> com
          <b>mais de 17 movimentos</b> tendem a demorar, é necessário aumentar o nª máximo de
          iterações.
        </span>
        <span v-if="type == 'bestA*' && !done">
          <b>**Atenção**</b> desafios em <u>Melhor A*</u> com <b>mais de 24 movimentos</b> podem
          levar vários minutos para encontrar a solução.
        </span>
      </v-row>
      <v-row justify="center" class="mt-6" style="font-size: 1.2rem"
        ><v-col align="center" cols="6" sm="6" md="4" lg="4" xl="3"
          ><Resolution
            v-if="done"
            :resolution="resolution"
            :meta="meta"
            :finalPath="finalPath"
            @nextStep="handleNextStepResolution"
            @previousStep="handlePreviousStepResolution"/></v-col
      ></v-row>
      <v-row v-if="done" justify="center" class="mt-8">
        <v-col
          cols="12"
          class="my-0 pa-0"
          justify="center"
          style="display:flex; justify-content: center"
        >
          <v-btn
            :disabled="!done"
            small
            rounded
            outlined
            elevation="0"
            color="green lighten-1"
            class="white--text ml-3"
            @click="showPathInfo = !showPathInfo"
          >
            {{ showPathInfo ? 'Esconder Caminho' : 'Mostrar Caminho' }}
          </v-btn>
          <v-btn
            :disabled="iterator > 2000"
            small
            rounded
            outlined
            elevation="0"
            color="purple lighten-2"
            class="white--text ml-3"
            @click="showFrontierInfo = !showFrontierInfo"
            >{{
              showFrontierInfo ? 'Esconder Fronteira/Visitados' : 'Mostrar Fronteira/Visitados'
            }}</v-btn
          >
        </v-col>
      </v-row>
      <!-- RESULTADO DETALHES -->
      <v-row v-if="done" justify="center" class="mt-12 mb-2" style="user-select: none;">
        <span class="mx-2"> <b>Máx Nodos na Fronteira:</b> {{ maxFrontier }} </span>
        <span class="mx-2"><b>Nodos Criados:</b> {{ createdNodes }}</span>
        <span class="mx-2"><b>Nodos Visitados:</b> {{ visitedNodes }}</span>
      </v-row>

      <!-- CAMINHO -->
      <PathResolution v-if="showPathInfo" :finalPath="finalPath" :meta="meta" :type="type" />

      <!-- FRONTEIRA E VISITADOS -->
      <ShowInfo
        v-if="showFrontierInfo"
        :frontier="frontier"
        :historyVisited="historyVisited"
        :meta="meta"
        :type="type"
      />
      <!-- Dialog informativo -->
      <v-dialog v-model="alert" retain-focus persistent width="500px">
        <v-card class="pa-6" align="center">
          <v-icon style="transform: scale(1.3);" class="mb-3" x-large color="yellow darken-2">
            mdi-alert
          </v-icon>
          <p>
            <span>
              <b>Ops...</b> o algoritmo não conseguiu encontrar uma solução para esse desafio :(
              <br />
            </span>
          </p>
          <v-btn
            rounded
            class="mt-4 white--text"
            elevation="0"
            color="yellow darken-2"
            @click="alert = false"
          >
            Ok
          </v-btn>
        </v-card>
      </v-dialog>
      <!-- customPuzzle -->
      <v-dialog v-model="customDialog" retain-focus persistent width="500px">
        <v-card class="pa-6" align="center">
          <v-icon style="transform: scale(1.3);" class="mb-3" x-large color="green lighten-2"
            >mdi-puzzle-edit</v-icon
          >
          <br />
          <span style="font-size: 1.2rem"><b>Desafio Personalizado</b></span>
          <br />
          <br />
          <p>
            <span
              >Para representar a casa sem número digite 0 no lugar.<br />Um
              <b>exemplo válido</b> de desafio é "123405786" <br
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
          <v-btn rounded class="mt-4" elevation="0" outlined @click="customDialog = false"
            >Cancelar</v-btn
          >
        </v-card>
      </v-dialog>
      <!-- Loading -->
      <Loading v-if="loading" />
    </v-card>
  </v-container>
</template>

<script>
import Meta from './Meta.vue'
import Challenge from './Challenge.vue'
import PathResolution from './PathResolution.vue'
import ShowInfo from './ShowInfo.vue'
import Resolution from './Resolution.vue'
import Loading from './Loading.vue'

export default {
  name: 'EightPuzzle',
  components: {
    Meta,
    Challenge,
    PathResolution,
    ShowInfo,
    Resolution,
    Loading
  },
  data: () => ({
    // VARIÁVEIS GLOBAIS
    start: false,
    showPathInfo: false,
    showFrontierInfo: false,
    type: 'bestA*',
    typeOptions: [
      { type: 'uniformCost', label: 'Custo Uniforme' },
      { type: 'a*', label: 'A* - Heurística Simples' },
      { type: 'bestA*', label: 'Melhor A*' }
    ],
    challengeByAlgorithm: [],
    challenges: [
      {
        label: 'Exemplo - 2 movimentos',
        puzzle: [
          ['1', '2', '3'],
          ['4', '', '5'],
          ['7', '8', '6']
        ]
      },
      {
        label: 'Exemplo - 6 movimentos',
        puzzle: [
          ['1', '6', '2'],
          ['4', '', '3'],
          ['7', '5', '8']
        ]
      },
      {
        label: 'Exemplo - 9 movimentos',
        puzzle: [
          ['1', '8', '2'],
          ['', '4', '3'],
          ['7', '6', '5']
        ]
      },
      {
        label: 'Exemplo - 11 movimentos',
        puzzle: [
          ['1', '2', '3'],
          ['', '6', '4'],
          ['7', '8', '5']
        ]
      },
      {
        label: 'Exemplo - 17 movimentos',
        puzzle: [
          ['4', '1', '2'],
          ['', '8', '7'],
          ['6', '3', '5']
        ]
      },
      {
        label: 'Exemplo - 22 movimentos',
        puzzle: [
          ['', '1', '2'],
          ['3', '4', '5'],
          ['6', '7', '8']
        ]
      },
      {
        label: 'Exemplo - 24 movimentos',
        puzzle: [
          ['6', '5', '1'],
          ['3', '', '8'],
          ['2', '7', '4']
        ]
      },
      {
        label: 'Exemplo - 27 movimentos',
        puzzle: [
          ['4', '7', '5'],
          ['', '2', '1'],
          ['3', '6', '8']
        ]
      },
      {
        label: 'Exemplo - 31 movimentos',
        puzzle: [
          ['6', '4', '7'],
          ['8', '5', ''],
          ['3', '2', '1']
        ]
      }
    ],
    selectedChallenge: [],
    defaultCost: 1,
    meta: [
      ['1', '2', '3'],
      ['4', '5', '6'],
      ['7', '8', '']
    ],
    visitedNodes: 0,
    createdNodes: 0,
    frontier: [],
    maxFrontier: 0,
    historyVisited: [],
    finalPath: [],
    visited: new Set(), // guardar os nodos visitados em .toString() para comparação
    frontierSet: new Set(), // lookup O(1) para verificar se puzzle já está na fronteira
    nodeIdCounter: 0,
    done: false,
    resolution: [],
    iterator: 0,
    alert: false,
    customDialog: false,
    newCustomPuzzle: '',
    errorMessage: '',
    loading: false,
    timeToDone: 0
  }),

  created() {
    this.addChallenge()
    this.selectedChallenge = this.challenges[0].puzzle
    const initialNode = {
      cost: 0,
      puzzle: this.selectedChallenge,
      puzzleKey: this.selectedChallenge.toString(),
      id: [this.nodeIdCounter++],
      heuristic1: this.useHeuristic1(this.selectedChallenge),
      heuristic2: this.useHeuristic2(this.selectedChallenge)
    }
    this.frontier.push(initialNode)
    this.frontierSet.add(initialNode.puzzleKey)
    this.maxFrontier = 1
  },

  watch: {
    //FRONTEND
    //Observa o tipo de busca escolhido pelo usuário e atualiza os desafios exemplos e iteraçoes
    type() {
      if (this.type == 'uniformCost') {
        this.challengeByAlgorithm = []
        this.addChallenge([0, 1, 2, 3])

        let check = 0
        this.challengeByAlgorithm.forEach(item => {
          if (item.puzzle.toString() === this.selectedChallenge.toString()) check++
        })

        if (check == 0) this.selectedChallenge = this.challenges[0].puzzle
      }
      if (this.type == 'a*') {
        this.challengeByAlgorithm = []
        this.addChallenge([0, 1, 2, 3, 4])

        let check = 0
        this.challengeByAlgorithm.forEach(item => {
          if (item.puzzle.toString() === this.selectedChallenge.toString()) check++
        })

        if (check == 0) this.selectedChallenge = this.challenges[0].puzzle
      }
      if (this.type == 'bestA*') {
        this.challengeByAlgorithm = []
        this.addChallenge()
      }
    },

    //ao selecionar o desafio, ele calcula dados iniciais e envia a fila da fronteira - é o o ponto de partida para a busca
    selectedChallenge() {
      this.frontier = []
      this.frontierSet.clear()
      const initialNode = {
        cost: 0,
        puzzle: this.selectedChallenge,
        puzzleKey: this.selectedChallenge.toString(),
        id: [this.nodeIdCounter++],
        heuristic1: this.useHeuristic1(this.selectedChallenge),
        heuristic2: this.useHeuristic2(this.selectedChallenge)
      }
      this.frontier.push(initialNode)
      this.frontierSet.add(initialNode.puzzleKey)
      this.maxFrontier = 1
    }
  },

  methods: {
    addChallenge(challenges = 'all') {
      const type = typeof challenges
      if (type === 'string' && challenges === 'all') {
        const availableChallenges = this.challenges.length
        for (let i = 0; i < availableChallenges; i++)
          this.challengeByAlgorithm.push(this.challenges[i])
      } else if (Array.isArray(challenges) && challenges.every(v => typeof v === 'number')) {
        challenges.forEach(v => {
          this.challengeByAlgorithm.push(this.challenges[v])
        })
      }
    },

    /* FRONTEND MÉTODOS - métodos referentes a interação com a tela e usuário */
    customPuzzle() {
      this.customDialog = true
    },

    //permite digitar apenas: 012345678 - no Puzzle personalziado
    validation($event) {
      let keyCode = $event.keyCode ? $event.keyCode : $event.which
      // only allow numbers
      if (keyCode < 48 || keyCode > 56) {
        $event.preventDefault()
      }
    },

    // valida o valor do Puzzle informado pelo usuario ao personalizar
    validationCustomPuzzle() {
      let set = new Set()
      let check = 0
      let numbers = this.newCustomPuzzle.toString()
      for (let i = 0; i < numbers.length; i++) {
        if (!set.has(numbers[i])) {
          check++
          set.add(numbers[i])
        }
      }
      if (check == 9 && numbers !== '123456780') {
        this.errorMessage = ''
        this.selectedChallenge = [
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

        this.challengeByAlgorithm.push({
          label: `Personalizado - ${numbers}`,
          puzzle: this.selectedChallenge
        })
        this.customDialog = false
      } else {
        this.errorMessage = 'Puzzle personalizado inválido.'
      }
    },

    // ao clicar no botão reset, zera todas as informações
    handleReset() {
      this.start = false
      this.selectedChallenge = this.challenges[0].puzzle
      this.frontier = []
      this.frontierSet.clear()
      this.nodeIdCounter = 0
      this.maxFrontier = 0
      const initialNode = {
        cost: 0,
        puzzle: this.selectedChallenge,
        puzzleKey: this.selectedChallenge.toString(),
        id: [this.nodeIdCounter++],
        heuristic1: this.useHeuristic1(this.selectedChallenge),
        heuristic2: this.useHeuristic2(this.selectedChallenge)
      }
      this.frontier.push(initialNode)
      this.frontierSet.add(initialNode.puzzleKey)
      this.visitedNodes = 0
      this.createdNodes = 0
      this.done = false
      this.visited.clear()
      this.historyVisited = []
      this.finalPath = []
      this.showPathInfo = false
      this.showFrontierInfo = false
      this.resolution = []
      this.iterator = 0
      this.newCustomPuzzle = ''
      this.errorMessage = ''
    },

    handleNextStepResolution() {
      for (let i = 0; i < this.finalPath.length; i++) {
        if (
          this.resolution.puzzle.toString() === this.finalPath[i].puzzle.toString() &&
          i + 1 < this.finalPath.length
        ) {
          this.resolution = this.finalPath[i + 1]
          return
        }
      }
    },
    handlePreviousStepResolution() {
      for (let i = 0; i < this.finalPath.length; i++) {
        if (
          this.resolution.puzzle.toString() === this.finalPath[i].puzzle.toString() &&
          i - 1 >= 0
        ) {
          this.resolution = this.finalPath[i - 1]
          return
        }
      }
    },

    // Clica no botão "Resolver" chama os métodos de lógica do algoritmo
    async handleStart() {
      this.loading = true
      this.start = true

      let timeStart = new Date().getTime()

      try {
        await new Promise(resolve => {
          setTimeout(() => {
            const metaKey = this.meta.toString()

            //enquanto o primeiro elemento da fronteira for diferente do objetivo, o laço é realizado
            while (this.frontier[0].puzzleKey !== metaKey) {
              this.iterator++
              let coordenates = this.foundEmptySpace(this.frontier[0])
              this.checkMovements(coordenates, this.frontier[0])
              if (this.frontier.length > this.maxFrontier) this.maxFrontier = this.frontier.length
            }

            // se o primeiro item da fronteira for igual ao objetivo ele encerra a busca e retorna os resultados
            if (this.frontier[0].puzzleKey === metaKey) {
              this.done = true
              this.visited.add(this.frontier[0].puzzleKey)
              this.frontierSet.delete(this.frontier[0].puzzleKey)
              this.historyVisited.push(this.frontier[0])
              this.geraFinalPath()
              this.resolution = this.finalPath[0]
              this.visitedNodes++
              this.frontier.splice(0, 1)
            }

            resolve()
          }, 0)
        })
      } catch (e) {
        this.alert = true
      } finally {
        this.calcSpendedTime(timeStart)
        this.loading = false
      }
    },

    calcSpendedTime(timeStart) {
      let elapsedTime = ''
      let timeEnd = new Date().getTime()
      let finalTime = timeEnd - timeStart // tempo decorrido em milisegundos
      this.timeToDone = finalTime

      if (finalTime > 1000) {
        let minutes = Math.floor(finalTime / 60000)
        let seconds = ((finalTime % 60000) / 1000).toFixed(0)
        if (minutes > 0)
          elapsedTime =
            minutes + ' minuto(s) e ' + (seconds < 10 ? '0' : '') + seconds + 'segundo(s)'
        if (minutes == 0) elapsedTime = seconds + ' segundo(s)'
      } else {
        elapsedTime = `${finalTime} milisegundos.`
      }

      console.log(`Total de iterações: ${this.iterator}`)
      console.log(`Tempo de execução de todas iterações: ${elapsedTime}`)
    },

    // Métodos para o ALGORITMO

    // Retorna o valor de custo usado para ordenação, conforme o tipo de busca
    getCostValue(node) {
      if (this.type === 'uniformCost') return node.cost
      if (this.type === 'a*') return node.cost + node.heuristic1
      return node.cost + node.heuristic2
    },

    // Insere nodo na fronteira mantendo ordem crescente via binary search - O(log n)
    insertOrdered(node) {
      const value = this.getCostValue(node)
      let low = 0
      let high = this.frontier.length
      while (low < high) {
        const mid = (low + high) >> 1
        if (this.getCostValue(this.frontier[mid]) <= value) {
          low = mid + 1
        } else {
          high = mid
        }
      }
      this.frontier.splice(low, 0, node)
    },

    // procura onde está espaço em branco da matriz e retorna seu indice
    foundEmptySpace(firstFrontier) {
      const x = firstFrontier.puzzle.findIndex(r => r.includes(''))
      const y = firstFrontier.puzzle[x]?.findIndex(c => c === '')
      return { x, y }
    },

    /* 1- checa os movimentos possiveis de serem realizados,

    2- método: checkPuzzleAlreadyCreated() -> checa se o nodo criado com o movimento(up,down,left,right) já está na fronteira ou na lista de nodos visitados

    3- adiciona o nodo a lista fronteira
    */

    checkMovements(coordenate, challenge) {
      const directions = [
        { dx: -1, dy: 0 }, // UP
        { dx: 1, dy: 0 }, // DOWN
        { dx: 0, dy: -1 }, // LEFT
        { dx: 0, dy: 1 } // RIGHT
      ]

      for (const { dx, dy } of directions) {
        const nx = coordenate.x + dx
        const ny = coordenate.y + dy
        if (nx < 0 || nx > 2 || ny < 0 || ny > 2) continue

        // Clone apenas o puzzle (não o objeto inteiro)
        const newPuzzle = challenge.puzzle.map(row => [...row])
        newPuzzle[coordenate.x][coordenate.y] = newPuzzle[nx][ny]
        newPuzzle[nx][ny] = ''

        const puzzleKey = newPuzzle.toString()
        if (!this.checkPuzzleAlreadyCreated(puzzleKey)) continue

        const newNode = {
          cost: challenge.cost + 1,
          puzzle: newPuzzle,
          puzzleKey,
          id: [...challenge.id, this.nodeIdCounter++],
          heuristic1: this.type === 'a*' ? this.useHeuristic1(newPuzzle) : 0,
          heuristic2: this.type === 'bestA*' ? this.useHeuristic2(newPuzzle) : 0
        }

        this.frontierSet.add(puzzleKey)
        this.insertOrdered(newNode)
        this.createdNodes++
      }

      // Move nodo atual da fronteira para visitados
      const current = this.frontier[0]
      this.visited.add(current.puzzleKey)
      this.frontierSet.delete(current.puzzleKey)
      this.historyVisited.push(current)
      this.visitedNodes++
      this.frontier.splice(0, 1)
    },

    /*recebe um toString da Matriz que foi criada com o movimento (up,down,left,right) e verifica se esse puzzle já foi criado. Verifica nos Nodos já visitados e também nos que estão na fronteira*/

    checkPuzzleAlreadyCreated(puzzleKey) {
      return !this.visited.has(puzzleKey) && !this.frontierSet.has(puzzleKey)
    },

    // Heuristica A* simples - recebe uma matriz e compara com a META, cada 'pastilha' no lugar errado incrementa em 1
    useHeuristic1(matriz) {
      let heuristic = 0

      if (matriz[0][0] !== this.meta[0][0]) heuristic++
      if (matriz[0][1] !== this.meta[0][1]) heuristic++
      if (matriz[0][2] !== this.meta[0][2]) heuristic++
      if (matriz[1][0] !== this.meta[1][0]) heuristic++
      if (matriz[1][1] !== this.meta[1][1]) heuristic++
      if (matriz[1][2] !== this.meta[1][2]) heuristic++
      if (matriz[2][0] !== this.meta[2][0]) heuristic++
      if (matriz[2][1] !== this.meta[2][1]) heuristic++
      if (matriz[2][2] !== this.meta[2][2]) heuristic++

      return heuristic
    },

    //Melhor Heurística - recebe uma matriz e compara com a META, usando distância de Manhattan
    useHeuristic2(matriz) {
      let heuristic = 0

      const goalMap = {
        1: [0, 0],
        2: [0, 1],
        3: [0, 2],
        4: [1, 0],
        5: [1, 1],
        6: [1, 2],
        7: [2, 0],
        8: [2, 1]
      }

      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          const value = matriz[i][j]

          if (value === '') continue

          const [goalI, goalJ] = goalMap[value]

          heuristic += Math.abs(i - goalI) + Math.abs(j - goalJ)
        }
      }

      return heuristic
    },

    //método auxiliar para gerar o final Path do resultado encontrado - usa Map para lookup O(1)
    geraFinalPath() {
      const idToNode = new Map()
      this.historyVisited.forEach(node => {
        idToNode.set(node.id[node.id.length - 1], node)
      })
      const lastNode = this.historyVisited[this.historyVisited.length - 1]
      this.finalPath = lastNode.id.map(id => idToNode.get(id))
    }
  }
}
</script>

<style></style>
