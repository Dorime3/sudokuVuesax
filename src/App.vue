<template>
  <div>
    <NavBar/>
    <div class="sudoku-wrapper">
      <div class="grid-sudoku">
        <div v-for="(row, rowIdx) in table" :key="rowIdx" class="grid-row">
          <div v-for="(cell, colIdx) in row" :key="colIdx" class="grid-cell">
            <input type="text" :key="colIdx" v-model="table[rowIdx][colIdx]" class="grid-cell-editor"/>
          </div>
        </div>
      </div>
      <div style="display: flex">
        <vs-button
            color="rgb(59,222,200)"
            gradient
            @click="activeDialog = !activeDialog"
            :disabled="!isShowLogs">
          Показать логи
        </vs-button>
        <vs-button gradient danger @click="clearTable">Очистить поле</vs-button>
        <vs-button gradient @click="resolveSudoku">Решить судоку</vs-button>
      </div>
      <vs-dialog blur v-model="activeDialog">
        <template #header>
          <h4>
            Отслеживание логов
          </h4>
        </template>
        <div>
          {{ logs }}
        </div>
      </vs-dialog>
    </div>
    <div style="display: flex; justify-content: end; color:#ff9292; font-size: 12px">
      @created by Родион
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import NavBar from "@/components/NavBar";

export default {
  components: {NavBar},
  data() {
    return {
      table: [
        ["5", "3", "", "", "7", "", "", "", ""],
        ["6", "", "", "1", "9", "5", "", "", ""],
        ["", "9", "8", "", "", "", "", "6", ""],
        ["8", "", "", "", "6", "", "", "", "3"],
        ["4", "", "", "8", "", "3", "", "", "1"],
        ["7", "", "", "", "2", "", "", "", "6"],
        ["", "6", "", "", "", "", "2", "8", ""],
        ["", "", "", "4", "1", "9", "", "", "5"],
        ["", "", "", "", "8", "", "", "7", "9"]
      ],
      tableSize: 9,
      boxSize: 3,
      activeDialog: false,
      isShowLogs: false,
      logs: ''
    }
  },
  watch: {},
  methods: {
    findEmpty() {
      for (let r = 0; r < this.tableSize; r++) {
        for (let c = 0; c < this.tableSize; c++) {
          if (this.table[r][c].length > 1) {
            return 'error'
          }
          if (this.table[r][c] === '') {
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

      if (curPos === 'error') {
        return false
      }

      for (let i = 1; i <= this.tableSize; i++) {
        const curNum = i.toString();
        const isValid = this.validate(curNum, curPos);
        this.logs += `curPos, ${curPos}, curNum, ${curNum}, isValid, ${isValid}, `

        if (isValid) {
          const [x, y] = curPos;
          // this.table[x][y] = curNum;
          Vue.set(this.table[x], y, curNum)

          if (this.isSolved()) {
            return true
          }

          // this.table[x][y] = '.'
          Vue.set(this.table[x], y, '')
        }

      }

      return false
    },

    resolveSudoku() {
      // @todo сделать первичную валидацию на правильность ввода исходных данных
      // в строках, столбцах и боксах не должны повторяться цифры

      const result = this.isSolved()
      this.logs += `isResult, ${this.table}`
      this.isShowLogs = true;

      if (result) {
        this.$vs.notification({
          color: 'success',
          position: 'top-right',
          duration: '8000',
          title: 'Эксельсиор!',
          text: 'Решение для вашего судоку найдено'
        })
      } else {
        this.$vs.notification({
          color: 'danger',
          position: 'top-right',
          duration: '8000',
          title: 'Увы :(',
          text: 'Этот судоку не решаем.'
        })
      }
    },

    clearTable() {
      for (let r = 0; r < this.tableSize; r++) {
        for (let c = 0; c < this.tableSize; c++) {
          Vue.set(this.table[r], c, '')
        }
      }
    }
  }
}
</script>

<style>
.vs-notification__content__text p {
  font-size: 14px;
}

.sudoku-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: calc(100vh - 40px);
  flex-direction: column;
}

.grid-sudoku {
  background: #d6e9ff;
  display: table;
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

.grid-cell:hover {
  background: #afd4ff;
  border: 1px solid forestgreen;
}

.grid-cell-editor {
  background: transparent;
  border: none;
  width: 20px;
  height: 20px;
  font-family: 'Dosis', sans-serif;
  font-weight: bold;
  text-align: center;
  font-size: 18px;
  transition: all ease 1.0s;
}

.grid-cell-editor:focus {
  color: forestgreen;
}

@media screen and (max-width: 600px) {
  .grid-sudoku {
    border: 2px solid black;
  }

  .grid-sudoku > div:nth-child(3), .grid-sudoku > div:nth-child(6) {
    border-bottom: 2px solid black;
  }

  .grid-row > div:nth-child(3), .grid-row > div:nth-child(6) {
    border-right: 2px solid black;
  }

  .grid-cell {
    padding: 5px;
  }

  .grid-cell-editor {
    width: 14px;
    height: 10px;
    font-size: 12px;
  }
}
</style>
