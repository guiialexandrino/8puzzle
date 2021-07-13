<template>
  <v-container>
    <v-row>
      <v-col class="my-1" cols="12" align="center">
        <h1 class="display-2 font-weight-bold mb-3">
          8-PUZZLE
        </h1>
        <p class="subheading font-weight-regular">
          Trabalho disciplina <b>INE 5633</b> - Sistemas inteligentes.<br />
          <span style="font-size:0.9rem"
            >Desenvolvido por: <b>Guilherme Alexandrino</b></span
          >
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
            <Challenge :selectedChallenge="selectedChallenge" />
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
    <v-row justify="center" class="mt-6" style="font-size: 1.2rem"
      ><v-col align="center" cols="6" sm="6" md="4" lg="4" xl="3"
        ><Resolution
          v-if="done"
          :resolution="resolution"
          :meta="meta"
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
          :disabled="!done"
          small
          rounded
          outlined
          elevation="0"
          color="purple lighten-2"
          class="white--text ml-3"
          @click="showFrontierInfo = !showFrontierInfo"
          >{{
            showFrontierInfo
              ? 'Esconder Fronteira/Visitados'
              : 'Mostrar Fronteira/Visitados'
          }}</v-btn
        >
      </v-col>
    </v-row>
    <!-- RESULTADO DETALHES -->
    <v-row v-if="done" justify="center" class="mt-12">
      <span class="mx-2"><b>Nodos na Fronteira:</b> {{ frontier.length }}</span>
      <span class="mx-2"><b>Nodos Criados:</b> {{ createdNodes }}</span>
      <span class="mx-2"><b>Nodos Visitados:</b> {{ visitedNodes }}</span>
    </v-row>

    <!-- CAMINHO -->
    <PathResolution
      v-if="showPathInfo"
      :finalPath="finalPath"
      :meta="meta"
      :type="type"
    />

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
        <v-icon
          style="transform: scale(1.3);"
          class="mb-3"
          x-large
          color="yellow darken-2"
          >mdi-alert</v-icon
        >
        <p>
          <span
            ><b>Ops...</b> parece que esse desafio <b>vai demorar muito :(</b
            ><br />Tente utilizando <b>outro algoritmo</b>, ok?</span
          >
        </p>
        <v-btn
          rounded
          class="mt-4 white--text"
          elevation="0"
          color="yellow darken-2"
          @click="alert = false"
          >Ok</v-btn
        >
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import Meta from './Meta.vue';
import Challenge from './Challenge.vue';
import PathResolution from './PathResolution.vue';
import ShowInfo from './ShowInfo.vue';
import Resolution from './Resolution.vue';

export default {
  name: 'EightPuzzle',
  components: { Meta, Challenge, PathResolution, ShowInfo, Resolution },
  data: () => ({
    start: false,
    showPathInfo: false,
    showFrontierInfo: false,
    type: 'uniformCost',
    typeOptions: [
      { type: 'uniformCost', label: 'Custo Uniforme' },
      { type: 'a*', label: 'A* - Heurística Simples' },
      { type: 'bestA*', label: 'Melhor A*' },
    ],
    //Selecionar exemplo
    challengeByAlgorithm: [],
    challenges: [
      {
        label: 'Exemplo - 2 movimentos',
        puzzle: [
          ['1', '2', '3'],
          ['4', '', '5'],
          ['7', '8', '6'],
        ],
      },
      {
        label: 'Exemplo - 6 movimentos',
        puzzle: [
          ['1', '6', '2'],
          ['4', '', '3'],
          ['7', '5', '8'],
        ],
      },
      {
        label: 'Exemplo - 9 movimentos',
        puzzle: [
          ['1', '8', '2'],
          ['', '4', '3'],
          ['7', '6', '5'],
        ],
      },
      {
        label: 'Exemplo - 11 movimentos',
        puzzle: [
          ['1', '2', '3'],
          ['', '6', '4'],
          ['7', '8', '5'],
        ],
      },
      {
        label: 'Exemplo - 17 movimentos',
        puzzle: [
          ['4', '1', '2'],
          ['', '8', '7'],
          ['6', '3', '5'],
        ],
      },
      {
        label: 'Exemplo - 22 movimentos',
        puzzle: [
          ['', '1', '2'],
          ['3', '4', '5'],
          ['6', '7', '8'],
        ],
      },
      {
        label: 'Exemplo - 24 movimentos',
        puzzle: [
          ['6', '5', '1'],
          ['3', '', '8'],
          ['2', '7', '4'],
        ],
      },
    ],
    selectedChallenge: [],
    defaultCost: 1,
    meta: [
      ['1', '2', '3'],
      ['4', '5', '6'],
      ['7', '8', ''],
    ],
    visitedNodes: 0,
    createdNodes: 0,
    frontier: [],
    historyVisited: [],
    finalPath: [],
    visited: new Set(), // guardar os nodos visitados em .toString() para comparação
    generatedIds: new Set(),
    done: false,
    resolution: [],
    iterator: 0,
    alert: false,
  }),
  created() {
    this.challengeByAlgorithm.push(this.challenges[0]);
    this.challengeByAlgorithm.push(this.challenges[1]);
    this.challengeByAlgorithm.push(this.challenges[2]);
    this.challengeByAlgorithm.push(this.challenges[3]);
    this.selectedChallenge = this.challenges[0].puzzle;
    this.frontier.push({
      cost: 0,
      puzzle: this.selectedChallenge,
      id: [this.geraStringAleatoria(3)],
      heuristic1: this.useHeuristic1(this.selectedChallenge),
      heuristic2: this.useHeuristic2(this.selectedChallenge),
    });
  },
  watch: {
    type() {
      if (this.type == 'uniformCost') {
        this.challengeByAlgorithm = [];
        this.challengeByAlgorithm.push(this.challenges[0]);
        this.challengeByAlgorithm.push(this.challenges[1]);
        this.challengeByAlgorithm.push(this.challenges[2]);
        this.challengeByAlgorithm.push(this.challenges[3]);

        let check = 0;
        this.challengeByAlgorithm.forEach((item) => {
          if (item.puzzle.toString() === this.selectedChallenge.toString())
            check++;
        });

        if (check == 0) this.selectedChallenge = this.challenges[0].puzzle;
      }
      if (this.type == 'a*') {
        this.challengeByAlgorithm = [];
        this.challengeByAlgorithm.push(this.challenges[0]);
        this.challengeByAlgorithm.push(this.challenges[1]);
        this.challengeByAlgorithm.push(this.challenges[2]);
        this.challengeByAlgorithm.push(this.challenges[3]);
        this.challengeByAlgorithm.push(this.challenges[4]);

        let check = 0;
        this.challengeByAlgorithm.forEach((item) => {
          if (item.puzzle.toString() === this.selectedChallenge.toString())
            check++;
        });

        if (check == 0) this.selectedChallenge = this.challenges[0].puzzle;
      }
      if (this.type == 'bestA*') {
        this.challengeByAlgorithm = [];
        this.challengeByAlgorithm.push(this.challenges[0]);
        this.challengeByAlgorithm.push(this.challenges[1]);
        this.challengeByAlgorithm.push(this.challenges[2]);
        this.challengeByAlgorithm.push(this.challenges[3]);
        this.challengeByAlgorithm.push(this.challenges[4]);
        this.challengeByAlgorithm.push(this.challenges[5]);
        this.challengeByAlgorithm.push(this.challenges[6]);
      }
    },
    selectedChallenge() {
      this.frontier = [];
      this.frontier.push({
        cost: 0,
        puzzle: this.selectedChallenge,
        id: [this.geraStringAleatoria(3)],
        heuristic1: this.useHeuristic1(this.selectedChallenge),
        heuristic2: this.useHeuristic2(this.selectedChallenge),
      });
    },
  },

  methods: {
    handleReset() {
      this.start = false;
      // this.selectedChallenge = this.challenges[0].puzzle;
      this.frontier = [];
      this.frontier.push({
        cost: 0,
        puzzle: this.selectedChallenge,
        id: [this.geraStringAleatoria(3)],
        heuristic1: this.useHeuristic1(this.selectedChallenge),
        heuristic2: this.useHeuristic2(this.selectedChallenge),
      });
      this.visitedNodes = 0;
      this.createdNodes = 0;
      this.done = false;
      this.visited.clear();
      this.historyVisited = [];
      this.finalPath = [];
      this.showPathInfo = false;
      this.showFrontierInfo = false;
      this.resolution = [];
      this.iterator = 0;
    },

    handleNextStepResolution() {
      for (let i = 0; i < this.finalPath.length; i++) {
        if (
          this.resolution.puzzle.toString() ===
            this.finalPath[i].puzzle.toString() &&
          i + 1 < this.finalPath.length
        ) {
          this.resolution = this.finalPath[i + 1];
          return;
        }
      }
    },
    handlePreviousStepResolution() {
      for (let i = 0; i < this.finalPath.length; i++) {
        if (
          this.resolution.puzzle.toString() ===
            this.finalPath[i].puzzle.toString() &&
          i - 1 >= 0
        ) {
          this.resolution = this.finalPath[i - 1];
          return;
        }
      }
    },

    handleStart() {
      this.start = true;
      while (this.frontier[0].puzzle.toString() !== this.meta.toString()) {
        this.iterator++;
        let coordenates = this.foundEmptySpace(this.frontier[0]);
        this.checkMovements(coordenates, this.frontier[0]);
        if (this.iterator > 2000) {
          this.alert = true;
          return;
        }
        this.lowestCost();
      }
      if (this.frontier[0].puzzle.toString() === this.meta.toString()) {
        // concluiu o objetivo!
        this.done = true;
        this.visited.add(this.frontier[0].puzzle.toString());
        this.historyVisited.push(this.frontier[0]);
        this.geraFinalPath();
        this.resolution = this.finalPath[0];
        this.visitedNodes++;
        this.frontier.splice(0, 1); // deleta o primeiro da fronteira
      }
    },

    // Métodos para o ALGORITMO

    // lowestCost -> ordena a lista da fronteira de acordo com o menor custo
    lowestCost() {
      if (this.type == 'uniformCost')
        this.frontier.sort((a, b) => {
          if (a.cost > b.cost) return 1;
          if (a.cost < b.cost) return -1;
          return 0;
        });
      if (this.type == 'a*')
        this.frontier.sort((a, b) => {
          if (a.cost + a.heuristic1 > b.cost + b.heuristic1) return 1;
          if (a.cost + a.heuristic1 < b.cost + b.heuristic1) return -1;
          return 0;
        });
      if (this.type == 'bestA*')
        this.frontier.sort((a, b) => {
          if (a.cost + a.heuristic2 > b.cost + b.heuristic2) return 1;
          if (a.cost + a.heuristic2 < b.cost + b.heuristic2) return -1;
          return 0;
        });
    },

    foundEmptySpace(firstFrontier) {
      let x, y;
      firstFrontier.puzzle.forEach((line, index) => {
        line.forEach((item, index2) => {
          if (!item) {
            x = index;
            y = index2;
          }
        });
      });
      return { x: x, y: y };
    },

    checkMovements(coordenate, challenge) {
      let nodeUp, nodeLeft, nodeRight, nodeDown;
      if (coordenate.x > 0) {
        //UP
        nodeUp = JSON.parse(JSON.stringify(challenge));
        nodeUp.puzzle[coordenate.x][coordenate.y] =
          nodeUp.puzzle[coordenate.x - 1][coordenate.y];
        nodeUp.puzzle[coordenate.x - 1][coordenate.y] = '';
        nodeUp.cost++;
        nodeUp.id.push(this.geraStringAleatoria(3));

        //se utilizar a heuristica a* simples
        if (this.type == 'a*')
          nodeUp.heuristic1 = this.useHeuristic1(nodeUp.puzzle);

        //se utilizar a heuristica bestA* simples
        if (this.type == 'bestA*')
          nodeUp.heuristic2 = this.useHeuristic2(nodeUp.puzzle);

        if (this.checkPuzzleAlreadyCreated(nodeUp.puzzle.toString())) {
          this.frontier.push(nodeUp);
          this.createdNodes++;
        }
      }
      if (coordenate.x < 2) {
        //Down
        nodeDown = JSON.parse(JSON.stringify(challenge));
        nodeDown.puzzle[coordenate.x][coordenate.y] =
          nodeDown.puzzle[coordenate.x + 1][coordenate.y];
        nodeDown.puzzle[coordenate.x + 1][coordenate.y] = '';
        nodeDown.cost++;
        nodeDown.id.push(this.geraStringAleatoria(3));

        //se utilizar a heuristica a* simples
        if (this.type == 'a*')
          nodeDown.heuristic1 = this.useHeuristic1(nodeDown.puzzle);

        //se utilizar a heuristica bestA* simples
        if (this.type == 'bestA*')
          nodeDown.heuristic2 = this.useHeuristic2(nodeDown.puzzle);

        if (this.checkPuzzleAlreadyCreated(nodeDown.puzzle.toString())) {
          this.frontier.push(nodeDown);
          this.createdNodes++;
        }
      }
      if (coordenate.y > 0) {
        //Left
        nodeLeft = JSON.parse(JSON.stringify(challenge));
        nodeLeft.puzzle[coordenate.x][coordenate.y] =
          nodeLeft.puzzle[coordenate.x][coordenate.y - 1];
        nodeLeft.puzzle[coordenate.x][coordenate.y - 1] = '';
        nodeLeft.cost++;
        nodeLeft.id.push(this.geraStringAleatoria(3));

        //se utilizar a heuristica a* simples
        if (this.type == 'a*')
          nodeLeft.heuristic1 = this.useHeuristic1(nodeLeft.puzzle);

        //se utilizar a heuristica bestA* simples
        if (this.type == 'bestA*')
          nodeLeft.heuristic2 = this.useHeuristic2(nodeLeft.puzzle);

        if (this.checkPuzzleAlreadyCreated(nodeLeft.puzzle.toString())) {
          this.frontier.push(nodeLeft);
          this.createdNodes++;
        }
      }
      if (coordenate.y < 2) {
        //Right
        nodeRight = JSON.parse(JSON.stringify(challenge));
        nodeRight.puzzle[coordenate.x][coordenate.y] =
          nodeRight.puzzle[coordenate.x][coordenate.y + 1];
        nodeRight.puzzle[coordenate.x][coordenate.y + 1] = '';
        nodeRight.cost++;
        nodeRight.id.push(this.geraStringAleatoria(3));

        //se utilizar a heuristica a* simples
        if (this.type == 'a*')
          nodeRight.heuristic1 = this.useHeuristic1(nodeRight.puzzle);

        //se utilizar a heuristica bestA* simples
        if (this.type == 'bestA*')
          nodeRight.heuristic2 = this.useHeuristic2(nodeRight.puzzle);

        if (this.checkPuzzleAlreadyCreated(nodeRight.puzzle.toString())) {
          this.frontier.push(nodeRight);
          this.createdNodes++;
        }
      }
      this.visited.add(this.frontier[0].puzzle.toString());
      this.historyVisited.push(this.frontier[0]);
      this.visitedNodes++;
      this.frontier.splice(0, 1); // deleta o primeiro da fronteira
    },

    checkPuzzleAlreadyCreated(toString) {
      //recebe um toString da Matriz que foi criada com o movimento (up,down,left,right) e verifica se esse puzzle já foi criado. Verifica nos Nodos já visitados e também nos que estão na fronteira
      let checkVisited = this.visited.has(toString);
      let checkFrontier = 0;
      this.frontier.forEach((item) => {
        if (item.puzzle.toString() == toString) checkFrontier++;
      });

      if (checkVisited == false && checkFrontier == 0) {
        return true;
      } else {
        return false;
      }
    },

    useHeuristic1(matriz) {
      //recebe uma matriz e compara com a META, cada 'pastilha' no lugar errado incrementa em 1
      let heuristic = 0;

      if (matriz[0][0] !== this.meta[0][0]) heuristic++;
      if (matriz[0][1] !== this.meta[0][1]) heuristic++;
      if (matriz[0][2] !== this.meta[0][2]) heuristic++;
      if (matriz[1][0] !== this.meta[1][0]) heuristic++;
      if (matriz[1][1] !== this.meta[1][1]) heuristic++;
      if (matriz[1][2] !== this.meta[1][2]) heuristic++;
      if (matriz[2][0] !== this.meta[2][0]) heuristic++;
      if (matriz[2][1] !== this.meta[2][1]) heuristic++;
      if (matriz[2][2] !== this.meta[2][2]) heuristic++;

      return heuristic;
    },

    useHeuristic2(matriz) {
      //recebe uma matriz e compara com a META, usando distância de Manhattan
      let heuristic = 0;
      const meta = [
        { value: 1, line: 0, column: 0 },
        { value: 2, line: 0, column: 1 },
        { value: 3, line: 0, column: 2 },
        { value: 4, line: 1, column: 0 },
        { value: 5, line: 1, column: 1 },
        { value: 6, line: 1, column: 2 },
        { value: 7, line: 2, column: 0 },
        { value: 8, line: 2, column: 1 },
        { value: '', line: 2, column: 2 },
      ];
      const array = [
        { value: matriz[0][0], line: 0, column: 0 },
        { value: matriz[0][1], line: 0, column: 1 },
        { value: matriz[0][2], line: 0, column: 2 },
        { value: matriz[1][0], line: 1, column: 0 },
        { value: matriz[1][1], line: 1, column: 1 },
        { value: matriz[1][2], line: 1, column: 2 },
        { value: matriz[2][0], line: 2, column: 0 },
        { value: matriz[2][1], line: 2, column: 1 },
        { value: matriz[2][2], line: 2, column: 2 },
      ];

      array.forEach((item) => {
        meta.forEach((metaItem) => {
          if (item.value == metaItem.value && item.value != '') {
            if (item.line !== metaItem.line)
              heuristic += Math.abs(item.line - metaItem.line);
            if (item.column !== metaItem.column)
              heuristic += Math.abs(item.column - metaItem.column);
          }
        });
      });

      return heuristic;
    },

    geraStringAleatoria(tamanho) {
      let check = false;
      while (!check) {
        var stringAleatoria = '';
        var caracteres =
          'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
        for (var i = 0; i < tamanho; i++) {
          stringAleatoria += caracteres.charAt(
            Math.floor(Math.random() * caracteres.length),
          );
        }
        if (!this.generatedIds.has(stringAleatoria)) {
          check = true;
          this.generatedIds.add(stringAleatoria);
        }
      }

      return stringAleatoria;
    },

    geraFinalPath() {
      let ids = this.historyVisited[this.historyVisited.length - 1];
      let path = [];
      ids.id.forEach((id) => {
        let result = this.historyVisited.filter((item) => {
          if (id == item.id[item.id.length - 1]) return item;
        });
        path.push(result[0]);
      });
      this.finalPath = path;
    },

    listaOrdenada(array, item) {
      let primeiroMaior = { value: '', index: '', checked: false };

      if (this.type == 'uniformCost') {
        for (let i = 0; i < array.length; i++) {
          if (array[i].cost > item.cost && !primeiroMaior.checked) {
            primeiroMaior.value = array[i].cost;
            primeiroMaior.index = i;
            primeiroMaior.checked = true;
            break;
          }
        }
      }

      if (!primeiroMaior.value) primeiroMaior.index = array.length; // adiciona no começo da array
      array.splice(primeiroMaior.index, 0, item);
    },
  },
};
</script>
