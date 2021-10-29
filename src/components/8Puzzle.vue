<template>
  <v-container>
    <!-- <v-row>
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
        <v-row justify="center" no-gutters class="mt-8 mb-4"
          ><v-icon class="pr-2 pb-5" small>mdi-cog</v-icon
          ><span>
            <b>Nº máximo de iterações: </b>
          </span>
          <v-select
            :disabled="type == 'bestA*'"
            style="max-width:5%; min-width: 80px"
            class="pl-2 ma-0"
            v-model="iterations"
            :items="iterationOptions"
            dense
          />
        </v-row>
      </v-col>
    </v-row> -->
    <v-card class="py-8 px-10" elevation="2">
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
          <v-row justify="center" no-gutters class="mt-8 mb-4"
            ><v-icon class="pr-2 pb-5" small>mdi-cog</v-icon
            ><span>
              <b>Nº máximo de iterações: </b>
            </span>
            <v-select
              :disabled="type == 'bestA*'"
              style="max-width:5%; min-width: 80px"
              class="pl-2 ma-0"
              v-model="iterations"
              :items="iterationOptions"
              dense
            />
          </v-row>
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
                @customPuzzle="customPuzzle"
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
          <b>mais de 11 movimentos</b> tendem a demorar, é necessário aumentar o
          nª máximo de iterações.
        </span>
        <span v-if="type == 'a*' && !done">
          <b>**Atenção**</b> desafios em <u>A* Heuristica Simples</u> com
          <b>mais de 17 movimentos</b> tendem a demorar, é necessário aumentar o
          nª máximo de iterações.
        </span>
        <span v-if="type == 'bestA*' && !done">
          <b>**Atenção**</b> desafios em <u>Melhor A*</u> com
          <b>mais de 24 movimentos</b> podem levar vários minutos para encontrar
          a solução.
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
              showFrontierInfo
                ? 'Esconder Fronteira/Visitados'
                : 'Mostrar Fronteira/Visitados'
            }}</v-btn
          >
        </v-col>
      </v-row>
      <!-- RESULTADO DETALHES -->
      <v-row
        v-if="done"
        justify="center"
        class="mt-12 mb-2"
        style="user-select: none;"
      >
        <span class="mx-2">
          <b>Máx Nodos na Fronteira:</b> {{ maxFrontier }}
        </span>
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
              ><b>Ops...</b> a busca atingiu o <b>limite máx de iterações</b> e
              não conseguiu encontrar uma solução para esse desafio :( <br
            /></span>
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
      <!-- customPuzzle -->
      <v-dialog v-model="customDialog" retain-focus persistent width="500px">
        <v-card class="pa-6" align="center">
          <v-icon
            style="transform: scale(1.3);"
            class="mb-3"
            x-large
            color="green lighten-2"
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
          <v-btn
            rounded
            class="mt-4"
            elevation="0"
            outlined
            @click="customDialog = false"
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
import Meta from './Meta.vue';
import Challenge from './Challenge.vue';
import PathResolution from './PathResolution.vue';
import ShowInfo from './ShowInfo.vue';
import Resolution from './Resolution.vue';
import Loading from './Loading.vue';

export default {
  name: 'EightPuzzle',
  components: {
    Meta,
    Challenge,
    PathResolution,
    ShowInfo,
    Resolution,
    Loading,
  },
  data: () => ({
    // VARIÁVEIS GLOBAIS
    start: false,
    showPathInfo: false,
    showFrontierInfo: false,
    type: 'uniformCost',
    typeOptions: [
      { type: 'uniformCost', label: 'Custo Uniforme' },
      { type: 'a*', label: 'A* - Heurística Simples' },
      { type: 'bestA*', label: 'Melhor A*' },
    ],
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
      {
        label: 'Exemplo - 27 movimentos',
        puzzle: [
          ['4', '7', '5'],
          ['', '2', '1'],
          ['3', '6', '8'],
        ],
      },
      {
        label: 'Exemplo - 31 movimentos',
        puzzle: [
          ['6', '4', '7'],
          ['8', '5', ''],
          ['3', '2', '1'],
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
    maxFrontier: 0,
    historyVisited: [],
    finalPath: [],
    visited: new Set(), // guardar os nodos visitados em .toString() para comparação
    generatedIds: new Set(),
    done: false,
    resolution: [],
    iterator: 0,
    iterations: 2000,
    iterationOptions: [2000, 3000, 4000, 5000, 10000, 30000],
    alert: false,
    customDialog: false,
    newCustomPuzzle: '',
    errorMessage: '',
    elapsedTime: '',
    loading: false,
    timeToDone: 0,
  }),

  created() {
    //FRONTEND
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
    this.maxFrontier = 1;
  },

  watch: {
    //FRONTEND
    //Observa o tipo de busca escolhido pelo usuário e atualiza os desafios exemplos e iteraçoes
    type() {
      if (this.type == 'uniformCost') {
        this.challengeByAlgorithm = [];
        this.iterations = 2000;
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
        this.iterations = 2000;
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
        this.iterations = 30000;
        this.challengeByAlgorithm.push(this.challenges[0]);
        this.challengeByAlgorithm.push(this.challenges[1]);
        this.challengeByAlgorithm.push(this.challenges[2]);
        this.challengeByAlgorithm.push(this.challenges[3]);
        this.challengeByAlgorithm.push(this.challenges[4]);
        this.challengeByAlgorithm.push(this.challenges[5]);
        this.challengeByAlgorithm.push(this.challenges[6]);
        this.challengeByAlgorithm.push(this.challenges[7]);
        this.challengeByAlgorithm.push(this.challenges[8]);
      }
    },

    //ao selecionar o desafio, ele calcula dados iniciais e envia a fila da fronteira - é o o ponto de partida para a busca
    selectedChallenge() {
      this.frontier = [];
      this.frontier.push({
        cost: 0,
        puzzle: this.selectedChallenge,
        id: [this.geraStringAleatoria(3)],
        heuristic1: this.useHeuristic1(this.selectedChallenge),
        heuristic2: this.useHeuristic2(this.selectedChallenge),
      });
      this.maxFrontier = 1;
    },
  },

  methods: {
    /* FRONTEND MÉTODOS - métodos referentes a interação com a tela e usuário */

    customPuzzle() {
      this.customDialog = true;
    },

    //permite digitar apenas: 012345678 - no Puzzle personalziado
    validation($event) {
      let keyCode = $event.keyCode ? $event.keyCode : $event.which;
      // only allow numbers
      if (keyCode < 48 || keyCode > 56) {
        $event.preventDefault();
      }
    },

    // valida o valor do Puzzle informado pelo usuario ao personalizar
    validationCustomPuzzle() {
      let set = new Set();
      let check = 0;
      let numbers = this.newCustomPuzzle.toString();
      for (let i = 0; i < numbers.length; i++) {
        if (!set.has(numbers[i])) {
          check++;
          set.add(numbers[i]);
        }
      }
      if (check == 9 && numbers !== '123456780') {
        this.errorMessage = '';
        this.selectedChallenge = [
          [
            numbers[0] !== '0' ? numbers[0] : '',
            numbers[1] !== '0' ? numbers[1] : '',
            numbers[2] !== '0' ? numbers[2] : '',
          ],
          [
            numbers[3] !== '0' ? numbers[3] : '',
            numbers[4] !== '0' ? numbers[4] : '',
            numbers[5] !== '0' ? numbers[5] : '',
          ],
          [
            numbers[6] !== '0' ? numbers[6] : '',
            numbers[7] !== '0' ? numbers[7] : '',
            numbers[8] !== '0' ? numbers[8] : '',
          ],
        ];

        this.challengeByAlgorithm.push({
          label: `Personalizado - ${numbers}`,
          puzzle: this.selectedChallenge,
        });
        this.customDialog = false;
      } else {
        this.errorMessage = 'Puzzle personalizado inválido.';
      }
    },

    // ao clicar no botão reset, zera todas as informações
    handleReset() {
      this.start = false;
      this.selectedChallenge = this.challenges[0].puzzle;
      this.frontier = [];
      this.maxFrontier = 0;
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
      this.newCustomPuzzle = '';
      this.errorMessage = '';
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

    // Clica no botão "Resolver" chama os métodos de lógica do algoritmo

    handleStart() {
      this.loading = true;
      this.start = true;

      setTimeout(() => {
        let timeStart = new Date().getTime();

        //enquanto o primeiro elemento da fronteira for diferente do objetivo, o laço é realizado
        while (this.frontier[0].puzzle.toString() !== this.meta.toString()) {
          this.iterator++;

          let coordenates = this.foundEmptySpace(this.frontier[0]);

          this.checkMovements(coordenates, this.frontier[0]);

          if (this.iterator > this.iterations) {
            this.loading = false;
            this.alert = true;
            return;
          }

          this.lowestCost();

          if (this.frontier.length > this.maxFrontier)
            this.maxFrontier = this.frontier.length;
        }

        // se o primeiro item da fronteira for igual ao objetivo ele encerra a busca e retorna os resultados
        if (this.frontier[0].puzzle.toString() === this.meta.toString()) {
          this.done = true;
          this.visited.add(this.frontier[0].puzzle.toString());
          this.historyVisited.push(this.frontier[0]);
          this.geraFinalPath();
          this.resolution = this.finalPath[0];
          this.visitedNodes++;
          this.frontier.splice(0, 1); // deleta o primeiro da fronteira
        }

        //Calcular o tempo de execução, pode ser visto no console do navegador (apertando f12)
        let timeEnd = new Date().getTime();
        let finalTime = timeEnd - timeStart; // tempo decorrido em milisegundos
        this.timeToDone = finalTime;
        if (finalTime > 1000) {
          let minutes = Math.floor(finalTime / 60000);
          let seconds = ((finalTime % 60000) / 1000).toFixed(0);
          if (minutes > 0)
            this.elapsedTime =
              minutes +
              ' minuto(s) e ' +
              (seconds < 10 ? '0' : '') +
              seconds +
              'segundo(s)';
          if (minutes == 0) this.elapsedTime = seconds + ' segundo(s)';
        } else {
          this.elapsedTime = `${finalTime} milisegundos.`;
        }
        console.log(`Total de iterações: ${this.iterator}`);
        console.log(
          `Tempo de execução de todas iterações: ${this.elapsedTime}`,
        );

        this.loading = false;
      }, 0);
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

    // procura onde está espaço em branco da matriz e retorna seu indice
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

    /* 1- checa os movimentos possiveis de serem realizados,

    2- método: checkPuzzleAlreadyCreated() -> checa se o nodo criado com o movimento(up,down,left,right) já está na fronteira ou na lista de nodos visitados

    3- adiciona o nodo a lista fronteira
    */

    checkMovements(coordenate, challenge) {
      let nodeUp, nodeLeft, nodeRight, nodeDown;

      //UP
      if (coordenate.x > 0) {
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

      //Down
      if (coordenate.x < 2) {
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

      //Left
      if (coordenate.y > 0) {
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

      //Right
      if (coordenate.y < 2) {
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

      /* utiliza a função SET do JS, adiciona o atual primeiro da fila, para os visitados.*/
      this.visited.add(this.frontier[0].puzzle.toString());

      /* adiciona na lista de visitados o atual primeiro da fila*/
      this.historyVisited.push(this.frontier[0]);

      this.visitedNodes++; //incrementa o numero de nodos visitados
      this.frontier.splice(0, 1); // deleta o atual primeiro da fronteira
    },

    /*recebe um toString da Matriz que foi criada com o movimento (up,down,left,right) e verifica se esse puzzle já foi criado. Verifica nos Nodos já visitados e também nos que estão na fronteira*/

    checkPuzzleAlreadyCreated(toString) {
      //verifica no SET se o nodo já está na lista de visitados;
      let checkVisited = this.visited.has(toString);

      //verifica se o nodo não está na fronteira
      let checkFrontier = 0;
      this.frontier.forEach((item) => {
        if (item.puzzle.toString() == toString) checkFrontier++;
      });

      //caso não esteja na fronteira e nem na lista de visitados, retorna TRUE
      if (checkVisited == false && checkFrontier == 0) {
        return true;
      } else {
        return false;
      }
    },

    // Heuristica A* simples - recebe uma matriz e compara com a META, cada 'pastilha' no lugar errado incrementa em 1
    useHeuristic1(matriz) {
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

    //Melhor Heurística - recebe uma matriz e compara com a META, usando distância de Manhattan
    useHeuristic2(matriz) {
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

    // método auxiliar para gerar o identificador de cada nodo
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

    //método auxiliar para gerar o final Path do resultado encontrado
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

    // tentativa de lista ordenada
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

<style></style>
