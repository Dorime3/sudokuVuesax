<template>
  <div>
    <NavBar/>
    <div class="sudoku-wrapper">
      <div style="display: flex; align-items: center">
        <h1 class="main-title">
          Решалкин судоку
        </h1>
        <div style="display: inline-block; margin: 5px 0 0 10px">
          <vs-tooltip shadow bottom not-hover v-model="showInfo">
            <vs-button
                icon
                warn
                gradient
                animation-type="rotate"
                @click="showInfo=!showInfo"
            >
              <i class='bx bx-cog'></i>
              <template #animate>
                <i class='bx bxs-cog'></i>
              </template>
            </vs-button>
            <template #tooltip>
              <div class="content-tooltip">
                <div class="body">
                  <vs-avatar circle size="80" ref="avatarContent">
                    <img
                        src="https://banner2.cleanpng.com/20180528/lyp/kisspng-citron-cactus-m-cartoon-flowerpot-prickly-5b0cb1d8b18e71.3609451315275586167273.jpg"
                        @load="closeLoading"
                        alt="">
                  </vs-avatar>
                </div>
                <footer>
                  Для поиска решения используется алгоритм DFS(в глубину), рекурсивно вызывающий функцию обратного
                  вызова для перебора всех подходящих значений для каждой ячейки таблицы.
                  Натягивая сову на глобус, я смог провалидировать вводимые данные и извещать об ошибке пользователя. Но
                  не все 😐. Поэтому есть вероятность "не выхода" из бесконечного цикла, что приведет к забитию
                  колстека, выделяемым вкладке
                  браузером.
                  <div style="display: flex; justify-content: center">
                    <vs-button
                        gradient
                        @click="showInfo=!showInfo">
                      Я понял, давай пока 👋
                    </vs-button>
                  </div>
                </footer>
              </div>
            </template>
          </vs-tooltip>
        </div>
      </div>
      <div class="grid-sudoku">
        <div v-for="(row, rowIdx) in table" :key="rowIdx" class="grid-row">
          <div v-for="(cell, colIdx) in row" :key="colIdx" class="grid-cell">
            <input type="text" :key="colIdx" :value="table[rowIdx][colIdx]" @input="setNumber($event ,rowIdx, colIdx)"
                   class="grid-cell-editor"/>
          </div>
        </div>
      </div>
      <div style="display: flex; margin-top: 5px">
        <vs-tooltip bottom color="rgb(59,222,200)">
          <vs-button
              color="rgb(59,222,200)"
              gradient
              @click="activeDialog = !activeDialog"
              :disabled="!isShowLogs">
            Показать логи
          </vs-button>
          <template #tooltip>
            Куча безполезной инфы
          </template>
        </vs-tooltip>
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
      <div class="promo-block">
        <PromoBlock/>
      </div>
    </div>
    <div style="display: flex; justify-content: space-between">
      <div style="width: 50px">
        <vs-switch dark v-model="darkTheme" @click="changeTheme">
          <template #circle>
            <i v-if="!darkTheme" class='bx bxs-sun'></i>
            <i v-else class='bx bxs-moon' style="color: #fff"></i>
          </template>
        </vs-switch>
      </div>
      <div style="font-size: 12px; display: flex; align-items: flex-end;">
        <p>@created by Родион</p>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import NavBar from "@/components/NavBar";
import PromoBlock from "@/components/PromoBlock";

export default {
  components: {PromoBlock, NavBar},
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
      globalLoading: null,
      isShowLogs: false,
      showInfo: false,
      darkTheme: false,
      logs: '',
      loadingAvatar: null
    }
  },
  watch: {
    showInfo(val) {
      setTimeout(() => {
        if (val) {
          this.loadingAvatar = this.$vs.loading({
            target: this.$refs.avatarContent,
            type: 'waves',
            color: '#d5397b',
          })
        }
      }, 0)
    },
  },
  mounted() {
    const currentTheme = localStorage.getItem('vsTheme')
    console.log('currentTheme', currentTheme)
    if (currentTheme !== 'light') {
      this.darkTheme = true;
      this.$vs.toggleTheme('light');
      document.body.classList.add('darken')
      document.body.style.background = '#18191c'
      document.body.style.color = '#fff'
      localStorage.setItem('vsTheme', 'dark')
    }
    // loader
    this.globalLoading = this.$vs.loading({
      type: 'waves',
      color: '#d5397b'
    })

    window.addEventListener('load', () => {
      setTimeout(() => {
        this.globalLoading.close();
      }, 1000)
    })
  },
  methods: {
    closeLoading() {
      this.loadingAvatar.close()
    },
    setNumber(e, row, col) {
      let currentVal = e.target.value[e.target.value.length - 1]
      currentVal = currentVal.match(/[1-9]/) ? currentVal : '';
      Vue.set(this.table[row], col, currentVal)
    },
    changeTheme() {
      this.toggleTheme(this.darkTheme ? 'dark' : 'light')
    },
    toggleTheme(theme) {
      this.$vs.toggleTheme(theme);
      if (theme === 'light') {
        document.body.classList.add('darken')
        document.body.style.background = '#18191c'
        document.body.style.color = '#fff'
      } else {
        document.body.classList.remove('darken')
        document.body.style.background = 'none'
        document.body.style.color = 'rgb(44, 62, 80)'
      }
    },
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
@import url('https://fonts.googleapis.com/css2?family=Amatic+SC:wght@700&display=swap');

body {
  margin-bottom: 0;
}

.vs-notification__content__text p {
  font-size: 14px;
}

.sudoku-wrapper {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  height: calc(100vh - 48px);
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

.promo-block {
  position: absolute;
  bottom: 0;
  right: 0;
}

.main-title {
  font-family: 'Amatic SC', cursive;
  font-size: 48px;
}

.content-tooltip .body {
  display: flex;
  align-items: flex-start;
  justify-content: center;
}

.content-tooltip .body .vs-avatar-content {
  margin-top: -50px;
  border: 3px solid transparent;
  box-shadow: 0 4px 15px 0 rgba(0, 0, 0, .1);
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
    width: 20px;
    height: 20px;
    font-size: 18px;
  }

  .main-title {
    font-family: 'Amatic SC', cursive;
    font-size: 36px;
  }
}


/*[vs-theme=dark] {*/
/*  --vs-theme-bg: #18191c;*/
/*  --vs-theme-color: #fff;*/
/*  --vs-theme-layout: #1e2023;*/
/*  --vs-theme-bg2: #141417;*/
/*  --vs-theme-code: #141417;*/
/*  --vs-theme-code2: #161619;*/
/*}*/
</style>
