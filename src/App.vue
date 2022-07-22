<template>
  <div>
    <vs-button @click="resolveSudoku">Решить судоку</vs-button>
    <transition name="fade">
      <div class="grid-sudoku">

        <div v-for="(row, rowIdx) in table" :key="rowIdx" class="grid-row">
          <div v-for="(cell, colIdx) in row" :key="colIdx" class="grid-cell">
            <transition-group tag="div" name="list-animation">
              <input type="text" :key="colIdx" v-model="table[rowIdx][colIdx]" class="grid-cell-editor"/>
            </transition-group>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import Vue from 'vue';

export default {
  data() {
    return {
      table: [
        ["5", "3", ".", ".", "7", ".", ".", ".", "."],
        ["6", ".", ".", "1", "9", "5", ".", ".", "."],
        [".", "9", "8", ".", ".", ".", ".", "6", "."],
        ["8", ".", ".", ".", "6", ".", ".", ".", "3"],
        ["4", ".", ".", "8", ".", "3", ".", ".", "1"],
        ["7", ".", ".", ".", "2", ".", ".", ".", "6"],
        [".", "6", ".", ".", ".", ".", "2", "8", "."],
        [".", ".", ".", "4", "1", "9", ".", ".", "5"],
        [".", ".", ".", ".", "8", ".", ".", "7", "9"]
      ],
      fuck: 0,
      tableSize: 9,
      boxSize: 3
    }
  },
  watch: {
    fuck(val) {
      console.log(val)
    }
  },
  methods: {
    findEmpty() {
      for (let r = 0; r < this.tableSize; r++) {
        for (let c = 0; c < this.tableSize; c++) {
          if (this.table[r][c] === '.') {
            return [r, c]
          }
        }
      }
      return null
    },

    validate(curNum, [r, c]) {
      // check rows & cols
      for (let i = 0; i < this.tableSize; i++) {
        if (this.table[i][c] === curNum && i !== r) {
          return false;
        }
        if (this.table[r][i] === curNum && i !== c) {
          return false
        }
      }
      // check box
      const boxRow = Math.floor(r / this.boxSize) * this.boxSize;
      const boxCol = Math.floor(c / this.boxSize) * this.boxSize;

      for (let i = boxRow; i < boxRow + this.boxSize; i++) {
        for (let j = boxCol; j < boxCol + this.boxSize; j++) {
          if (this.table[i][j] === curNum && i !== r && j !== c) {
            return false
          }
        }
      }
      return true
    },


    isSolved() {
      const curPos = this.findEmpty();

      if (curPos === null) {
        return true
      }

      for (let i = 1; i <= this.tableSize; i++) {
        const curNum = i.toString();
        const isValid = this.validate(curNum, curPos);
        console.log('curPos', curPos, 'curNum', curNum, 'isValid', isValid)

        if (isValid) {
          const [x, y] = curPos;
          // this.table[x][y] = curNum;
          Vue.set(this.table[x], y, curNum)

          if (this.isSolved()) {
            return true
          }

          // this.table[x][y] = '.'
          Vue.set(this.table[x], y, '.')
        }

      }

      return false
    },

    resolveSudoku() {
      this.isSolved()
      console.table(this.table)
    }
  }
}
</script>

<style scoped>
.grid-sudoku {
  display: table;
  background: white;
  border: 3px solid black;
}

.grid-sudoku > div:nth-child(3), .grid-sudoku > div:nth-child(6) {
  border-bottom: 3px solid black;
}

.grid-row > div:nth-child(3), .grid-row > div:nth-child(6) {
  border-right: 3px solid black;
}

.grid-cell {
  display: table-cell;
  padding: 10px;
  border: 1px solid gray;
}

.grid-cell-editor {
  border: none;
  width: 20px;
  height: 20px;
  font-family: 'Dosis', sans-serif;
  font-weight: bold;
  text-align: center;
  font-size: 18px;
  transition: all ease 1.0s;
}
</style>
