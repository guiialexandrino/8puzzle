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
              <Challenge
                :selectedChallenge="selectedChallenge"
                @customPuzzle="customDialog = true"
              />
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
          >
            Resolver
          </v-btn>
          <v-btn
            rounded
            elevation="0"
            color="red lighten-2"
            class="white--text ml-3"
            @click="handleReset"
          >
            Resetar
          </v-btn>
        </v-col>
      </v-row>

      <v-row justify="center" class="mt-6" style="font-size: 1.2rem">
        <v-col align="center" cols="6" sm="6" md="4" lg="4" xl="3">
          <Resolution
            v-if="done"
            :resolution="resolution"
            :meta="meta"
            :finalPath="finalPath"
            @nextStep="handleNextStepResolution"
            @previousStep="handlePreviousStepResolution"
        /></v-col>
      </v-row>

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

      <v-row v-if="done" justify="center" class="mt-12 mb-2" style="user-select: none;">
        <span class="mx-2"><b>Máx Nodos na Fronteira:</b> {{ maxFrontier }}</span>
        <span class="mx-2"><b>Nodos Criados:</b> {{ createdNodes }}</span>
        <span class="mx-2"><b>Nodos Visitados:</b> {{ visitedNodes }}</span>
      </v-row>

      <PathResolution v-if="showPathInfo" :finalPath="finalPath" :meta="meta" :type="type" />

      <ShowInfo
        v-if="showFrontierInfo"
        :frontier="frontier"
        :historyVisited="historyVisited"
        :meta="meta"
        :type="type"
      />

      <AlertDialog v-model="alert" />

      <CustomPuzzleDialog
        v-model="customDialog"
        @custom-puzzle-created="handleCustomPuzzleCreated"
      />

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
import AlertDialog from './AlertDialog.vue'
import CustomPuzzleDialog from './CustomPuzzleDialog.vue'
import challenges from '../utils/challenges'

export default {
  name: 'EightPuzzle',
  components: {
    Meta,
    Challenge,
    PathResolution,
    ShowInfo,
    Resolution,
    Loading,
    AlertDialog,
    CustomPuzzleDialog
  },

  data: () => ({
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
    challenges,
    selectedChallenge: [],
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

    // Estruturas preparadas para Bidirectional A*
    forwardFrontier: [],
    forwardVisited: new Set(),
    backwardFrontier: [],
    backwardVisited: new Set(),
    forwardHistoryVisited: [],
    backwardHistoryVisited: [],
    cameFromForward: new Map(),
    cameFromBackward: new Map(),
    gScoreForward: new Map(),
    gScoreBackward: new Map(),
    nodeIdCounter: 0,
    done: false,
    resolution: [],
    iterator: 0,
    alert: false,
    customDialog: false,
    loading: false
  }),

  created() {
    this.addChallenge()
    this.selectedChallenge = this.challenges[0].puzzle
    this.initializeBidirectionalStructures(this.selectedChallenge, this.meta)
  },

  watch: {
    type() {
      if (this.type == 'uniformCost') {
        this.challengeByAlgorithm = []
        this.addChallenge([0, 1, 2, 3, 4])
        this.ensureSelectedChallengeIsAvailable()
      }

      if (this.type == 'a*') {
        this.challengeByAlgorithm = []
        this.addChallenge([0, 1, 2, 3, 4, 5])
        this.ensureSelectedChallengeIsAvailable()
      }

      if (this.type == 'bestA*') {
        this.challengeByAlgorithm = []
        this.addChallenge()
      }
    },

    selectedChallenge() {
      this.initializeBidirectionalStructures(this.selectedChallenge, this.meta)
    }
  },

  methods: {
    buildBidirectionalRootNode(puzzle, targetPuzzle, direction) {
      return {
        direction,
        cost: 0,
        puzzle,
        puzzleKey: puzzle.toString(),
        heuristic1: this.computeHeuristic(puzzle, { mode: 'a*', targetPuzzle }),
        heuristic2: this.computeHeuristic(puzzle, { mode: 'bestA*', targetPuzzle })
      }
    },

    initializeBidirectionalStructures(startPuzzle, goalPuzzle) {
      this.forwardFrontier = []
      this.forwardVisited.clear()
      this.backwardFrontier = []
      this.backwardVisited.clear()
      this.forwardHistoryVisited = []
      this.backwardHistoryVisited = []
      this.cameFromForward = new Map()
      this.cameFromBackward = new Map()
      this.gScoreForward = new Map()
      this.gScoreBackward = new Map()

      const forwardRoot = this.buildBidirectionalRootNode(startPuzzle, goalPuzzle, 'forward')
      const backwardRoot = this.buildBidirectionalRootNode(goalPuzzle, startPuzzle, 'backward')

      this.forwardFrontier.push(forwardRoot)
      this.backwardFrontier.push(backwardRoot)
      this.cameFromForward.set(forwardRoot.puzzleKey, null)
      this.cameFromBackward.set(backwardRoot.puzzleKey, null)
      this.gScoreForward.set(forwardRoot.puzzleKey, 0)
      this.gScoreBackward.set(backwardRoot.puzzleKey, 0)
    },

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

    ensureSelectedChallengeIsAvailable() {
      const selectedChallengeKey = this.selectedChallenge.toString()
      const challengeExists = this.challengeByAlgorithm.some(
        item => item.puzzle.toString() === selectedChallengeKey
      )

      if (!challengeExists) this.selectedChallenge = this.challenges[0].puzzle
    },

    handleCustomPuzzleCreated(customChallenge) {
      this.selectedChallenge = customChallenge.puzzle
      this.challengeByAlgorithm.push(customChallenge)
    },

    handleReset() {
      this.start = false
      this.nodeIdCounter = 0
      this.maxFrontier = 0
      this.initializeBidirectionalStructures(this.selectedChallenge, this.meta)
      this.visitedNodes = 0
      this.createdNodes = 0
      this.done = false
      this.historyVisited = []
      this.finalPath = []
      this.showPathInfo = false
      this.showFrontierInfo = false
      this.resolution = []
      this.iterator = 0
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

    async handleStart() {
      this.loading = true
      this.start = true
      await this.yieldToBrowser()

      let timeStart = new Date().getTime()

      try {
        const metaKey = this.meta.toString()
        this.runBidirectionalSearch(metaKey)
      } catch (e) {
        this.alert = true
      } finally {
        this.calcSpendedTime(timeStart)
        this.loading = false
      }
    },

    yieldToBrowser() {
      return new Promise(resolve => setTimeout(resolve, 0))
    },

    calcSpendedTime(timeStart) {
      let elapsedTime = ''
      let timeEnd = new Date().getTime()
      let finalTime = timeEnd - timeStart // tempo decorrido em milisegundos

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

    runBidirectionalSearch(metaKey) {
      const startKey = this.selectedChallenge.toString()
      const iterationLimit = 100000

      // Sempre reinicia o estado dual antes de processar
      this.initializeBidirectionalStructures(this.selectedChallenge, this.meta)
      this.frontier = []
      this.historyVisited = []
      this.finalPath = []
      this.done = false
      this.createdNodes = 0
      this.visitedNodes = 0
      this.iterator = 0
      this.maxFrontier =
        this.forwardFrontier.length + this.backwardFrontier.length > 0
          ? this.forwardFrontier.length + this.backwardFrontier.length
          : 0

      if (startKey === metaKey) {
        this.finalPath = this.reconstructBidirectionalFinalPath(startKey)
        this.done = true
        this.resolution = this.finalPath[0]
        return
      }

      let bestPathCost = Infinity
      let bestMeetingKey = null

      while (this.forwardFrontier.length > 0 && this.backwardFrontier.length > 0) {
        this.iterator++
        if (this.iterator > iterationLimit) {
          throw new Error('Limite de iterações atingido na busca bidirecional')
        }

        const forwardTop = this.peekValidBidirectionalNode('forward')
        const backwardTop = this.peekValidBidirectionalNode('backward')
        if (!forwardTop || !backwardTop) break

        const forwardTopCost = this.getBidirectionalNodeScore(forwardTop)
        const backwardTopCost = this.getBidirectionalNodeScore(backwardTop)
        const lowerBound = Math.min(forwardTopCost, backwardTopCost)

        // Encerra quando o limite inferior não consegue melhorar o melhor encontro encontrado
        if (bestPathCost !== Infinity && lowerBound >= bestPathCost) break

        const sideToExpand = forwardTopCost <= backwardTopCost ? 'forward' : 'backward'
        const expansion = this.expandBidirectionalSide(sideToExpand)
        if (!expansion) continue

        if (expansion.meetingCost < bestPathCost) {
          bestPathCost = expansion.meetingCost
          bestMeetingKey = expansion.meetingKey
        }

        const currentCombinedFrontier = this.forwardFrontier.length + this.backwardFrontier.length
        if (currentCombinedFrontier > this.maxFrontier) {
          this.maxFrontier = currentCombinedFrontier
        }
      }

      if (!bestMeetingKey) {
        throw new Error('Algoritmo bidirecional não encontrou caminho até a meta')
      }

      this.finalPath = this.reconstructBidirectionalFinalPath(bestMeetingKey)
      this.done =
        this.finalPath.length > 0 && this.finalPath[this.finalPath.length - 1].puzzleKey === metaKey
      if (!this.done) {
        throw new Error('Reconstrução do caminho bidirecional não chegou na meta')
      }

      this.historyVisited = [...this.forwardHistoryVisited, ...this.backwardHistoryVisited]
      this.frontier = this.mergeBidirectionalFrontiersForUI()
      this.resolution = this.finalPath[0]
      this.visitedNodes = this.forwardHistoryVisited.length + this.backwardHistoryVisited.length
    },

    getBidirectionalNodeScore(node) {
      if (this.type === 'uniformCost') return node?.cost || 0
      if (this.type === 'a*') return (node?.cost || 0) + (node?.heuristic1 || 0)
      return (node?.cost || 0) + (node?.heuristic2 || 0)
    },

    insertOrderedOnFrontier(frontier, node, scoreGetter) {
      const value = scoreGetter.call(this, node)
      let low = 0
      let high = frontier.length
      while (low < high) {
        const mid = (low + high) >> 1
        if (scoreGetter.call(this, frontier[mid]) <= value) {
          low = mid + 1
        } else {
          high = mid
        }
      }
      frontier.splice(low, 0, node)
    },

    peekValidBidirectionalNode(side) {
      const frontier = side === 'forward' ? this.forwardFrontier : this.backwardFrontier
      const gScoreMap = side === 'forward' ? this.gScoreForward : this.gScoreBackward
      while (frontier.length > 0) {
        const node = frontier[0]
        const knownBest = gScoreMap.get(node.puzzleKey)
        if (knownBest === undefined || node.cost !== knownBest) {
          frontier.splice(0, 1)
          continue
        }
        return node
      }
      return null
    },

    popValidBidirectionalNode(side) {
      const frontier = side === 'forward' ? this.forwardFrontier : this.backwardFrontier
      const visitedSet = side === 'forward' ? this.forwardVisited : this.backwardVisited
      const history = side === 'forward' ? this.forwardHistoryVisited : this.backwardHistoryVisited
      const gScoreMap = side === 'forward' ? this.gScoreForward : this.gScoreBackward

      const node = this.peekValidBidirectionalNode(side)
      if (!node) return null

      frontier.splice(0, 1)
      visitedSet.add(node.puzzleKey)
      history.push(node)
      this.visitedNodes++

      const knownBest = gScoreMap.get(node.puzzleKey)
      if (knownBest === undefined || node.cost !== knownBest) return null
      return node
    },

    buildBidirectionalSearchNode(puzzle, cost, targetPuzzle, direction) {
      return {
        direction,
        cost,
        puzzle,
        puzzleKey: puzzle.toString(),
        id: [this.nodeIdCounter++],
        heuristic1: this.computeHeuristic(puzzle, { mode: 'a*', targetPuzzle }),
        heuristic2: this.computeHeuristic(puzzle, { mode: 'bestA*', targetPuzzle })
      }
    },

    getNeighborPuzzles(puzzle) {
      const position = this.foundEmptySpace({ puzzle })
      const directions = [
        { dx: -1, dy: 0 },
        { dx: 1, dy: 0 },
        { dx: 0, dy: -1 },
        { dx: 0, dy: 1 }
      ]
      const neighbors = []

      for (const { dx, dy } of directions) {
        const nx = position.x + dx
        const ny = position.y + dy
        if (nx < 0 || nx > 2 || ny < 0 || ny > 2) continue

        const newPuzzle = puzzle.map(row => [...row])
        newPuzzle[position.x][position.y] = newPuzzle[nx][ny]
        newPuzzle[nx][ny] = ''
        neighbors.push(newPuzzle)
      }

      return neighbors
    },

    expandBidirectionalSide(side) {
      const currentNode = this.popValidBidirectionalNode(side)
      if (!currentNode) return null

      const currentKey = currentNode.puzzleKey
      const currentCost = currentNode.cost

      const currentG = side === 'forward' ? this.gScoreForward : this.gScoreBackward
      const oppositeG = side === 'forward' ? this.gScoreBackward : this.gScoreForward
      const frontier = side === 'forward' ? this.forwardFrontier : this.backwardFrontier
      const cameFrom = side === 'forward' ? this.cameFromForward : this.cameFromBackward
      const visitedSet = side === 'forward' ? this.forwardVisited : this.backwardVisited
      const targetPuzzle = side === 'forward' ? this.meta : this.selectedChallenge
      const direction = side

      let bestMeetingCost = Infinity
      let bestMeetingKey = null

      if (oppositeG.has(currentKey)) {
        bestMeetingCost = currentCost + oppositeG.get(currentKey)
        bestMeetingKey = currentKey
      }

      const neighbors = this.getNeighborPuzzles(currentNode.puzzle)
      for (const neighborPuzzle of neighbors) {
        const neighborKey = neighborPuzzle.toString()
        const tentativeG = currentCost + 1
        const existingG = currentG.get(neighborKey)

        if (visitedSet.has(neighborKey) && existingG !== undefined && tentativeG >= existingG) {
          continue
        }

        if (existingG !== undefined && tentativeG >= existingG) continue

        currentG.set(neighborKey, tentativeG)
        cameFrom.set(neighborKey, currentKey)

        const neighborNode = this.buildBidirectionalSearchNode(
          neighborPuzzle,
          tentativeG,
          targetPuzzle,
          direction
        )
        this.insertOrderedOnFrontier(frontier, neighborNode, this.getBidirectionalNodeScore)
        this.createdNodes++

        if (oppositeG.has(neighborKey)) {
          const candidateCost = tentativeG + oppositeG.get(neighborKey)
          if (candidateCost < bestMeetingCost) {
            bestMeetingCost = candidateCost
            bestMeetingKey = neighborKey
          }
        }
      }

      return {
        meetingCost: bestMeetingCost,
        meetingKey: bestMeetingKey
      }
    },

    puzzleFromKey(key) {
      const values = key.split(',')
      return [
        [values[0], values[1], values[2]],
        [values[3], values[4], values[5]],
        [values[6], values[7], values[8]]
      ]
    },

    reconstructChain(cameFrom, endKey) {
      const chain = []
      let cursor = endKey
      while (cursor !== null && cursor !== undefined) {
        chain.push(cursor)
        cursor = cameFrom.get(cursor)
      }
      return chain
    },

    reconstructBidirectionalFinalPath(meetingKey) {
      const forwardChainMeetingToStart = this.reconstructChain(this.cameFromForward, meetingKey)
      const backwardChainMeetingToGoal = this.reconstructChain(this.cameFromBackward, meetingKey)

      const startToMeeting = [...forwardChainMeetingToStart].reverse()
      const meetingToGoal = backwardChainMeetingToGoal.slice(1)
      const fullKeyPath = [...startToMeeting, ...meetingToGoal]

      return fullKeyPath.map((key, index) => {
        const puzzle = this.puzzleFromKey(key)
        return {
          cost: index,
          puzzle,
          puzzleKey: key,
          id: Array.from({ length: index + 1 }, (_, i) => i),
          heuristic1: this.computeHeuristic(puzzle, { mode: 'a*', targetPuzzle: this.meta }),
          heuristic2: this.computeHeuristic(puzzle, { mode: 'bestA*', targetPuzzle: this.meta })
        }
      })
    },

    mergeBidirectionalFrontiersForUI() {
      return [...this.forwardFrontier, ...this.backwardFrontier].sort(
        (a, b) => this.getBidirectionalNodeScore(a) - this.getBidirectionalNodeScore(b)
      )
    },

    foundEmptySpace(firstFrontier) {
      const x = firstFrontier.puzzle.findIndex(r => r.includes(''))
      const y = firstFrontier.puzzle[x]?.findIndex(c => c === '')
      return { x, y }
    },

    getMisplacedTilesDistance(matriz, targetPuzzle) {
      let heuristic = 0

      if (matriz[0][0] !== '' && matriz[0][0] !== targetPuzzle[0][0]) heuristic++
      if (matriz[0][1] !== '' && matriz[0][1] !== targetPuzzle[0][1]) heuristic++
      if (matriz[0][2] !== '' && matriz[0][2] !== targetPuzzle[0][2]) heuristic++
      if (matriz[1][0] !== '' && matriz[1][0] !== targetPuzzle[1][0]) heuristic++
      if (matriz[1][1] !== '' && matriz[1][1] !== targetPuzzle[1][1]) heuristic++
      if (matriz[1][2] !== '' && matriz[1][2] !== targetPuzzle[1][2]) heuristic++
      if (matriz[2][0] !== '' && matriz[2][0] !== targetPuzzle[2][0]) heuristic++
      if (matriz[2][1] !== '' && matriz[2][1] !== targetPuzzle[2][1]) heuristic++
      if (matriz[2][2] !== '' && matriz[2][2] !== targetPuzzle[2][2]) heuristic++

      return heuristic
    },

    computeHeuristic(matriz, { mode = this.type, targetPuzzle = this.meta } = {}) {
      if (mode === 'a*') return this.getMisplacedTilesDistance(matriz, targetPuzzle)
      if (mode === 'bestA*') return this.getManhattanDistance(matriz, targetPuzzle)
      return 0
    },

    buildGoalMap(targetPuzzle) {
      const goalMap = {}
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          const value = targetPuzzle[i][j]
          if (value === '') continue
          goalMap[value] = [i, j]
        }
      }
      return goalMap
    },

    getManhattanDistance(matriz, targetPuzzle) {
      let heuristic = 0
      const goalMap = this.buildGoalMap(targetPuzzle)

      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          const value = matriz[i][j]

          if (value === '') continue

          const [goalI, goalJ] = goalMap[value]

          heuristic += Math.abs(i - goalI) + Math.abs(j - goalJ)
        }
      }

      return heuristic
    }
  }
}
</script>

<style></style>
