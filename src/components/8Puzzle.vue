<template>
  <v-container>
    <v-row>
      <v-col class="my-1" cols="12" align="center">
        <h1 class="display-2 font-weight-bold mb-3">
          8-PUZZLE
        </h1>
        <p class="subheading font-weight-regular">
          Trabalho disciplina <b>INE 5633</b> - Sistemas inteligentes.
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
            ></v-select>
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
          </v-col>
          <v-col cols="6" sm="6" md="4" lg="4" xl="3">
            <v-select
              :disabled="start"
              class="px-4"
              v-model="selectedChallenge"
              :items="challenges"
              item-text="label"
              item-value="puzzle"
            ></v-select>
            <!-- Mostra Meta -->
            <v-row
              justify="center"
              class="grey lighten-4"
              style="width:70%; font-size: 1.2rem"
            >
              <v-col cols="12" class="white"><b>Desafio</b></v-col>
              <v-col
                v-for="linha1 in selectedChallenge[0]"
                :key="linha1"
                cols="4"
                :class="linha1 == '' && 'red lighten-4'"
                >{{ linha1 }}
              </v-col>
              <v-col
                v-for="linha2 in selectedChallenge[1]"
                :key="linha2"
                cols="4"
                :class="linha2 == '' && 'red lighten-4'"
                >{{ linha2 }}
              </v-col>
              <v-col
                v-for="linha3 in selectedChallenge[2]"
                :key="linha3"
                cols="4"
                :class="linha3 == '' && 'red lighten-4'"
                >{{ linha3 }}
              </v-col>
            </v-row>
            <!-- Fim Meta -->
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
    <!-- Resultados -->
    <v-row class="mb-3">
      <v-col cols="12">
        <b>CAMINHO:</b>
        <v-row class="mt-1 green lighten-5" align="center">
          <v-col
            style="max-width:85%"
            align="center"
            cols="12"
            sm="6"
            md="6"
            lg="4"
            xl="3"
            v-for="(puzzle, index) in finalPath"
            :key="index"
          >
            <span
              v-for="(id, index) in puzzle.id"
              :key="index"
              style="font-size: 0.8rem"
            >
              {{ id }}
              <span v-if="index < puzzle.id.length - 1">-</span>
            </span>
            <div
              style="letter-spacing: 1.5rem; margin-top: 1rem"
              :class="
                index == 0 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[0][0] && 'transparent--text'">
                {{ puzzle.puzzle[0][0] ? `${puzzle.puzzle[0][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[0][1] && 'transparent--text'">
                {{ puzzle.puzzle[0][1] ? `${puzzle.puzzle[0][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[0][2] && 'transparent--text'">
                {{ puzzle.puzzle[0][2] ? `${puzzle.puzzle[0][2]}` : '0' }}
              </span>
            </div>
            <div
              style="letter-spacing: 1.5rem"
              :class="
                index == 0 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[1][0] && 'transparent--text'">
                {{ puzzle.puzzle[1][0] ? `${puzzle.puzzle[1][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[1][1] && 'transparent--text'">
                {{ puzzle.puzzle[1][1] ? `${puzzle.puzzle[1][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[1][2] && 'transparent--text'">
                {{ puzzle.puzzle[1][2] ? `${puzzle.puzzle[1][2]}` : '0' }}
              </span>
            </div>
            <div
              style="letter-spacing: 1.5rem"
              :class="
                index == 0 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[2][0] && 'transparent--text'">
                {{ puzzle.puzzle[2][0] ? `${puzzle.puzzle[2][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[2][1] && 'transparent--text'">
                {{ puzzle.puzzle[2][1] ? `${puzzle.puzzle[2][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[2][2] && 'transparent--text'">
                {{ puzzle.puzzle[2][2] ? `${puzzle.puzzle[2][2]}` : '0' }}
              </span>
            </div>
            <br />
            <span
              v-html="
                puzzle.cost == 0
                  ? `<b>Início</b>`
                  : `<b>Custo</b>: ${puzzle.cost}`
              "
            />
            <br />
          </v-col>
        </v-row>
      </v-col>
    </v-row>
    <v-row justify="center" class="my-7">
      <span class="mx-2"><b>Nodos na Fronteira:</b> {{ frontier.length }}</span>
      <span class="mx-2"><b>Nodos Criados:</b> {{ createdNodes }}</span>
      <span class="mx-2"><b>Nodos Visitados:</b> {{ visitedNodes }}</span>
    </v-row>
    <v-row class="mb-5">
      <v-col cols="6" class="grey lighten-4">
        <b>FRONTEIRA:</b>
        <v-row class="mt-1" align="center">
          <v-col
            style="max-width:85%"
            align="center"
            cols="12"
            sm="6"
            md="6"
            lg="4"
            xl="3"
            v-for="(puzzle, index) in frontier"
            :key="index"
          >
            <span
              v-for="(id, index) in puzzle.id"
              :key="index"
              style="font-size: 0.8rem"
            >
              {{ id }}
              <span v-if="index < puzzle.id.length - 1">-</span>
            </span>
            <div
              style="letter-spacing: 1.5rem; margin-top: 1rem"
              :class="
                index == 0 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[0][0] && 'transparent--text'">
                {{ puzzle.puzzle[0][0] ? `${puzzle.puzzle[0][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[0][1] && 'transparent--text'">
                {{ puzzle.puzzle[0][1] ? `${puzzle.puzzle[0][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[0][2] && 'transparent--text'">
                {{ puzzle.puzzle[0][2] ? `${puzzle.puzzle[0][2]}` : '0' }}
              </span>
            </div>
            <div
              style="letter-spacing: 1.5rem"
              :class="
                index == 0 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[1][0] && 'transparent--text'">
                {{ puzzle.puzzle[1][0] ? `${puzzle.puzzle[1][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[1][1] && 'transparent--text'">
                {{ puzzle.puzzle[1][1] ? `${puzzle.puzzle[1][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[1][2] && 'transparent--text'">
                {{ puzzle.puzzle[1][2] ? `${puzzle.puzzle[1][2]}` : '0' }}
              </span>
            </div>
            <div
              style="letter-spacing: 1.5rem"
              :class="
                index == 0 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[2][0] && 'transparent--text'">
                {{ puzzle.puzzle[2][0] ? `${puzzle.puzzle[2][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[2][1] && 'transparent--text'">
                {{ puzzle.puzzle[2][1] ? `${puzzle.puzzle[2][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[2][2] && 'transparent--text'">
                {{ puzzle.puzzle[2][2] ? `${puzzle.puzzle[2][2]}` : '0' }}
              </span>
            </div>
            <br />
            <span
              v-html="
                puzzle.cost == 0
                  ? `<b>Início</b>`
                  : `<b>Custo</b>: ${puzzle.cost}`
              "
            />
            <br />
          </v-col>
        </v-row>
      </v-col>
      <!-- Visitados -->
      <v-col cols="6" class="blue lighten-5">
        <b>VISITADOS:</b>
        <v-row class="mt-1" align="center">
          <v-col
            style="max-width:85%"
            align="center"
            cols="12"
            sm="6"
            md="6"
            lg="6"
            xl="3"
            v-for="(puzzle, index) in historyVisited"
            :key="index"
          >
            <span
              v-for="(id, index) in puzzle.id"
              :key="index"
              style="font-size: 0.8rem"
            >
              {{ id }}
              <span v-if="index < puzzle.id.length - 1">-</span>
            </span>
            <div
              style="letter-spacing: 1.5rem; margin-top: 1rem"
              :class="
                index == historyVisited.length - 1 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[0][0] && 'transparent--text'">
                {{ puzzle.puzzle[0][0] ? `${puzzle.puzzle[0][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[0][1] && 'transparent--text'">
                {{ puzzle.puzzle[0][1] ? `${puzzle.puzzle[0][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[0][2] && 'transparent--text'">
                {{ puzzle.puzzle[0][2] ? `${puzzle.puzzle[0][2]}` : '0' }}
              </span>
            </div>
            <div
              style="letter-spacing: 1.5rem"
              :class="
                index == historyVisited.length - 1 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[1][0] && 'transparent--text'">
                {{ puzzle.puzzle[1][0] ? `${puzzle.puzzle[1][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[1][1] && 'transparent--text'">
                {{ puzzle.puzzle[1][1] ? `${puzzle.puzzle[1][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[1][2] && 'transparent--text'">
                {{ puzzle.puzzle[1][2] ? `${puzzle.puzzle[1][2]}` : '0' }}
              </span>
            </div>
            <div
              style="letter-spacing: 1.5rem"
              :class="
                index == historyVisited.length - 1 &&
                  puzzle.puzzle.toString() == meta.toString() &&
                  'green lighten-4'
              "
            >
              <span :class="!puzzle.puzzle[2][0] && 'transparent--text'">
                {{ puzzle.puzzle[2][0] ? `${puzzle.puzzle[2][0]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[2][1] && 'transparent--text'">
                {{ puzzle.puzzle[2][1] ? `${puzzle.puzzle[2][1]}` : '0' }}
              </span>
              <span :class="!puzzle.puzzle[2][2] && 'transparent--text'">
                {{ puzzle.puzzle[2][2] ? `${puzzle.puzzle[2][2]}` : '0' }}
              </span>
            </div>
            <br />
            <span
              v-html="
                puzzle.cost == 0
                  ? `<b>Início</b>`
                  : `<b>Custo</b>: ${puzzle.cost}`
              "
            />
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'EightPuzzle',
  created() {
    this.selectedChallenge = this.challenges[0].puzzle;
    this.frontier.push({
      cost: 0,
      puzzle: this.selectedChallenge,
      id: [this.geraStringAleatoria(3)],
    });
  },
  watch: {
    selectedChallenge() {
      this.frontier = [];
      this.frontier.push({
        cost: 0,
        puzzle: this.selectedChallenge,
        id: [this.geraStringAleatoria(3)],
      });
    },
  },
  data: () => ({
    start: false,
    type: 'uniformCost',
    typeOptions: [
      { type: 'uniformCost', label: 'Custo Uniforme' },
      { type: 'a*', label: 'A*' },
      { type: 'bestA*', label: 'Melhor A*' },
    ],
    //Selecionar exemplo
    challenges: [
      {
        label: 'Exemplo 1',
        puzzle: [
          ['1', '2', ''],
          ['4', '5', '3'],
          ['7', '8', '6'],
        ],
      },
      {
        label: 'Exemplo 2',
        puzzle: [
          ['1', '2', '3'],
          ['4', '', '5'],
          ['7', '8', '6'],
        ],
      },
      {
        label: 'Exemplo 3',
        puzzle: [
          ['1', '8', '2'],
          ['', '4', '3'],
          ['7', '6', '5'],
        ],
      },
      {
        label: 'Exemplo 4',
        puzzle: [
          ['1', '6', '2'],
          ['4', '', '3'],
          ['7', '5', '8'],
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
  }),

  // Métodos FRONTEND
  methods: {
    handleReset() {
      this.start = false;
      this.selectedChallenge = this.challenges[0].puzzle;
      this.frontier = [];
      this.frontier.push({
        cost: 0,
        puzzle: this.selectedChallenge,
        id: [this.geraStringAleatoria(3)],
      });
      this.visitedNodes = 0;
      this.createdNodes = 0;
      this.done = false;
      this.visited.clear();
      this.historyVisited = [];
    },

    handleStart() {
      this.start = true;
      while (this.frontier[0].puzzle.toString() !== this.meta.toString()) {
        let coordenates = this.foundEmptySpace(this.frontier[0]);
        this.checkMovements(coordenates, this.frontier[0]);
        this.lowestCost();
      }
      if (this.frontier[0].puzzle.toString() === this.meta.toString()) {
        console.log('Primeiro da fronteira é igual ao OBJETIVO!!!');
        this.done = true;
        this.visited.add(this.frontier[0].puzzle.toString());
        this.historyVisited.push(this.frontier[0]);
        this.geraFinalPath();
        this.visitedNodes++;
        this.frontier.splice(0, 1); // deleta o primeiro da fronteira
      }

      // if (this.frontier[0].puzzle.toString() !== this.meta.toString()) {
      //   let coordenates = this.foundEmptySpace(this.frontier[0]);
      //   this.checkMovements(coordenates, this.frontier[0]);
      //   this.lowestCost();

      // } else {
      //   console.log('Primeiro da fronteira é igual ao OBJETIVO!!!');
      //   this.done = true;
      //   this.visited.add(this.frontier[0].puzzle.toString());
      //   this.historyVisited.push(this.frontier[0]);
      //   this.visitedNodes++;
      //   this.frontier.splice(0, 1); // deleta o primeiro da fronteira
      // }
    },

    // Métodos para o ALGORITMO

    // lowestCost -> ordena a lista da fronteira de acordo com o menor custo
    lowestCost() {
      this.frontier.sort((a, b) => {
        if (a.cost > b.cost) return 1;
        if (a.cost < b.cost) return -1;
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
  },
};
</script>
