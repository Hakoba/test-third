<template>
  <v-container>
    <v-layout text-xs-center wrap>
      <v-flex mb-5 xs12>
        <h2 class="headline font-weight-bold mb-3">Найди пару</h2>
        <v-layout justify-center>
          <v-card v-if="timer != null" class="card" style="background:#444" width="430px">
            <!-- выводим элементы на которые будем тыкать -->
            <v-btn
              class="square"
              v-for="(item, index) in squares" 
              :key="index"
              :class="{'disable-events': item.checked == true, 'enable-events': item.checked == false}"
              :style="{'background-color': item.checked == false ? '#fff' : item.color   }"
              @click="tryCheck(item)"
            ></v-btn>
          </v-card>
        </v-layout>
      </v-flex>
    </v-layout>
    <span>
      Время:
      {{ timerHours }}
      : {{ timerMinutes }}
      : {{ timerSeconds }}
    </span>
    <br>
    <v-btn @click="controlTimer">Начать</v-btn>
    <!-- POP UP -->
      <div class="text-xs-center">
        <v-dialog v-model="dialog" width="420">
          <v-card>
            <v-card-title class="headline grey lighten-2" primary-title>Поп ап</v-card-title>
            <v-card-text>
              Вы выиграли!
              <br>
              Время:
              {{ timerHours }}
              : {{ timerMinutes }}
              : {{ timerSeconds }}
            </v-card-text>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" flat @click="dialog = false">Ясн</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </div>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    squares: [], //массив объектов с "квдратами"
    last: null,  // хранит в себе последний нажатый квадратик
    timer: null, 
    time: 0, 
    runTimer: true, //состояние секндомера
    counterOfTrueAnswers: 0, // при наборе 8 стопает игру
    dialog: false //  состояние попапа
  }),
  mounted() {
    // Тут при маунтеде задаю 8 элеменетов с рандомным цветом, и копию к нему
    let counter = 0;
    let squares = new Array();
    for (let i = 0; i < 8; i++) {
      let color = this.randColor();
      squares = [
        {
          id: counter, //собственно  -  оригинал 
          checked: false,
          color: color
        },
        {
          id: counter + 1, // копия
          checked: false,
          color: color
        },
        ...squares
      ];
      counter = counter + 2;
    }
    this.squares = squares.sort(this.compareRandom);
  },
  methods: {
    controlTimer() { // 
      this.runTimer = !this.runTimer;

      if (this.runTimer) {
        clearInterval(this.timer);
        return;
      }
      this.timer = setInterval(() => {
        this.time++;
      }, 1000);
    },
    randColor() { // генерирует рандомный цвет
      var o = Math.round,
        r = Math.random,
        s = 255;
      return "rgb(" + o(r() * s) + "," + o(r() * s) + "," + o(r() * s) + ")";
    },
    compareRandom(a, b) { // вспомогательная функция для генерации цветов при маунтеде компонента
      return Math.random() - 0.5;
    },
    tryCheck(a) {    // основная функция игры
      if (this.last === null) { // если до этого не было нажато кнопки или нажато не правильно
        a.checked = !a.checked;
        this.last = a;
      } else {
        if (a.color == this.last.color) { // если совпадение то
          // console.log("норм");
          a.checked = true;
          this.last = null; // обнуляем  ласт пушед баттон
          this.counterOfTrueAnswers++; // увеличвае колличество сопадений
          if (this.counterOfTrueAnswers == 8) { //если 8 совпадений игра окончена
            this.controlTimer();
            this.dialog = true;  
          }
        } else { // не совпал цвет. 
          a.checked = true;
          // console.log("не норм");
          setTimeout(() => {
            this.last.checked = false;
            this.last = null;
            a.checked = false;
          }, 300);
        }
      }
    }
  },
  computed: {
    timerHours() {
      return Math.floor(this.timer / 3600) //высчитывем часы
        ? Math.floor(this.timer / 3600)
        : "0" + Math.floor(this.timer / 3600);
    },
    timerMinutes() { //высчитывем минуты
      let min = Math.floor((this.time % 3600) / 60);
      return min > 9 ? min : "0" + min;
    },
    timerSeconds() { 
      return this.time % 60 > 9 ? this.time % 60 : "0" + (this.time % 60);
    }
  }
};
</script>

<style>
.disable-events {
  pointer-events: none;
}
.enable-events {
  pointer-events: auto;
}
.card {
  padding: 15px !important;
}
.square {
  width: 40px;
  height: 40px;
  border: 2px solid #444;
  margin: 0 !important;
}
.white {
  background: white !important;
}
</style>
