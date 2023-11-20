<template>
  <div class="wrapper">
    <h1 class="wrapper__title">Simon</h1>
    <div class="scores">
      <h3 class="scores__current-score">Раунд : {{ score }}</h3>
      <p class="scores__message">{{ message }}</p>
      <div class="level block">
        <p class="level__label">Уровень сложности</p>
        <label
          v-for="level in ['Легкий', 'Средний', 'Сложный']"
          :key="level"
          class="level__item"
        >
          <input
            type="radio"
            v-model="selectedLevel"
            name="level"
            @click="updateTime(level)"
            :value="level"
            class="level__input"
          />
          {{ level }}
        </label>
      </div>
    </div>
    <div class="game">
      <div class="game-pad">
        <button class="pad pad-green" ref="1" @click="handleClick(1)"></button>
        <button class="pad pad-red" ref="2" @click="handleClick(2)"></button>
        <button class="pad pad-yellow" ref="3" @click="handleClick(3)"></button>
        <button class="pad pad-blue" ref="4" @click="handleClick(4)"></button>
      </div>
      <button class="game-start" @click="startGame">START</button>
    </div>
  </div>
</template>

<script>
import sound_green from "@/assets/sound_green.mp3";
import sound_red from "@/assets/sound_red.mp3";
import sound_yellow from "@/assets/sound_yellow.mp3";
import sound_blue from "@/assets/sound_blue.mp3";
import sound_end from "@/assets/sound_end.mp3";

export default {
  name: "SimonGame",
  data() {
    return {
      score: 0,
      isGameInProgress: false,
      selectedLevel: "Легкий",
      time: 1500,
      timeoutId: null,
      levelTime: {
        Легкий: "1500",
        Средний: "1000",
        Сложный: "400",
      },
      audio: {
        1: new Audio(sound_green),
        2: new Audio(sound_red),
        3: new Audio(sound_yellow),
        4: new Audio(sound_blue),
        5: new Audio(sound_end),
      },
      orderList: [],
      userList: [],
      message: "Нажмите на кнопку start!"
    };
  },
  methods: {
    handleClick(id) {
      this.audio[id].play();
      this.$refs[id].style.opacity = "0.5";
      this.$refs[id].style.transform = "scale(0.9)";
      setTimeout(() => {
        this.$refs[id].style.opacity = "1";
        this.$refs[id].style.transform = "scale(1)";
      }, 500);
      if (this.isGameInProgress) {
        clearTimeout(this.timeoutId);
        this.userList.push(id);
        this.checkUserList(this.userList.length - 1);
      }
    },

    startGame() {
      this.time = this.levelTime[this.selectedLevel];
      this.orderList = [];
      this.userList = [];
      this.startNextRound();
    },

    startNextRound() {
      this.userList = [];
      this.playOrderList();
    },

    playOrderList() {
      this.isGameInProgress = false;
      this.orderList.push(this.getRandomButtonId());
      this.message = "Слушайте!";

      this.orderList.forEach((id, i) => {
        setTimeout(() => {
          this.handleClick(id);
        }, this.levelTime[this.selectedLevel] * i);
      });
      setTimeout(() => {
        this.isGameInProgress = true;
        this.message = "Повторите!";
      }, this.orderList.length * this.levelTime[this.selectedLevel]);
    },

    getRandomButtonId() {
      const num = 1 + Math.random() * (4 + 1 - 1);
      return Math.floor(num);
    },

    checkUserList(index) {
      if (this.userList[index] !== this.orderList[index]) {
        return this.gameOver();
      }
      if (this.userList.length === this.orderList.length) {
        this.score++;
        setTimeout(() => {
          this.startNextRound();
        }, 1000);
      }
    },

    updateTime(selectedLevel) {
      this.time = this.levelTime[selectedLevel];
      this.message = "Нажмите на кнопку start!";
    },

    gameOver() {
      clearTimeout(this.timeoutId);
      this.score = 0;
      this.isGameInProgress = false;
      this.userList = [];
      this.orderList = [];
      this.audio[5].play();
      this.message = "Вы проиграли!";
    },
  },
};
</script>
