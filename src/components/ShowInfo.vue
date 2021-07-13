<template>
  <v-row class="my-8">
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
                ? `<b>Desafio</b>`
                : `<b>Custo</b>: ${puzzle.cost}`
            "
          />
          <br />
          <span v-if="type == 'a*'"
            ><b>Heuristica:</b> {{ puzzle.heuristic1 }}</span
          >
          <span v-if="type == 'bestA*'"
            ><b>Melhor Heuristica:</b> {{ puzzle.heuristic2 }}</span
          >
        </v-col>
      </v-row>
    </v-col>
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
                ? `<b>Desafio</b>`
                : `<b>Custo</b>: ${puzzle.cost}`
            "
          />
          <br />
          <span v-if="type == 'a*'"
            ><b>Heuristica:</b> {{ puzzle.heuristic1 }}</span
          >
          <span v-if="type == 'bestA*'"
            ><b>Melhor Heuristica:</b> {{ puzzle.heuristic2 }}</span
          >
        </v-col>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'ShowInfo',
  props: {
    frontier: {
      type: Array,
      default: () => {
        return [];
      },
    },
    historyVisited: {
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
    type: {
      type: String,
    },
  },
};
</script>

<style></style>
