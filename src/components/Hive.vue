<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import { useMainStore } from "../store";
import { shuffle } from "../utils";
import { useI18n } from "vue-i18n";
import en from "../locales/en.json";

const { t } = useI18n({
  inheritLocale: true,
  messages: {
    en,
  },
});

// `defineProps` is a compiler macro and no longer needs to be imported.
defineProps({
  ZIndex: Number,
});
const store = useMainStore();

const otherLetters = ref(
  store.availableLetters
    .split("")
    .filter((l: string) => l !== store.middleLetter)
);
let userGuess = ref("");

const onKeyPress = (e: KeyboardEvent) => {
  const pressedKey = e.key.toLowerCase();
  if (pressedKey === "enter")
    return submitGuess({ $t: t, guess: userGuess.value });
  if (["backspace", "delete"].includes(pressedKey)) {
    userGuess.value = userGuess.value.slice(0, -1);
    return false;
  }
  if (pressedKey.length === 1 && store.availableLetters.includes(pressedKey)) {
    userGuess.value += pressedKey;
    return true;
  }
};

const submitGuess = ({ $t, guess }: { $t: Function; guess: string }) => {
  userGuess.value = "";
  store.submitGuess({ $t, guess });
};

onMounted(() => {
  window.addEventListener("keyup", onKeyPress);
});
onUnmounted(() => {
  window.removeEventListener("keyup", onKeyPress);
});
</script>

<template>
  <div class="sb-controls" :style="`z-index: ${ZIndex}`">
    <div class="user-guess">
      <strong
        v-for="(letter, index) in userGuess"
        :class="{ 'middle-letter': letter === store.middleLetter }"
        :key="`user-guess-${index}`">
        {{ letter }}
      </strong>
    </div>

    <div class="hive">
      <svg
        class="hive-cell center"
        @click="userGuess += store.middleLetter"
        viewBox="0 0 120 104">
        <polygon
          class="cell-fill"
          points="0,52 30,0 90,0 120,52 90,104 30,104"
          :stroke="store.getColor"
          stroke-width="7.5" />
        <text class="cell-letter" x="50%" y="50%" dy="10.75%">
          {{ store.middleLetter }}
        </text>
      </svg>
      <svg
        v-for="(letter, index) in otherLetters"
        :key="index"
        @click="userGuess += letter"
        class="hive-cell outer"
        viewBox="0 0 120 104">
        <polygon
          class="cell-fill"
          points="0,52 30,0 90,0 120,52 90,104 30,104"
          :stroke="store.getColor"
          stroke-width="7.5" />
        <text class="cell-letter" x="50%" y="50%" dy="10.75%">
          {{ letter }}
        </text>
      </svg>
    </div>

    <div class="hive-actions">
      <button
        class="hive-action hive-action__delete sb-touch-button"
        style="margin-left: 0; min-width: 5.5em"
        @click="userGuess = userGuess.slice(0, -1)">
        {{ $t("Delete") }}
      </button>
      <button
        class="hive-action hive-action__shuffle sb-touch-button"
        @click="otherLetters = shuffle(otherLetters, Math.random())"></button>
      <button
        class="hive-action hive-action__submit sb-touch-button"
        style="min-width: 5.5em"
        @click="submitGuess({ $t, guess: userGuess })">
        {{ $t("Enter") }}
      </button>
    </div>
  </div>
</template>

<style scoped lang="scss">
@import "../assets/styles/_variables";

.user-guess {
  text-transform: uppercase;
  margin-bottom: 10px;
  height: 35px;
  font-weight: 700;
  font-size: 25px;
  .middle-letter {
    color: $bl-yellow;
  }
}
.sb-controls {
  /* put entire hive behind correctGuesses when table is expanded */
  max-width: 290px;
  position: absolute;
  top: 65%;
  left: 50%;
  transform: translate(-50%, -50%);
  opacity: 1;
  transition: opacity 150ms 200ms ease;
}
.hive {
  position: relative;
  width: 100%;
  padding-bottom: 100%;
}
.hive-cell {
  position: absolute;
  top: calc(100% / 3);
  left: 30%;
  width: 40%;
  height: calc(100% / 3);
}
.hive-cell .cell-fill {
  cursor: pointer;
  fill: $bl-grey;
  transition: all 100ms;
}
.hive .cell-letter {
  font-weight: 700;
  font-size: 30px;
  text-anchor: middle;
  text-transform: uppercase;
  pointer-events: none;
}
.hive-cell:first-child .cell-fill {
  cursor: pointer;
  fill: $bl-yellow;
  transition: all 100ms;
}
.hive-cell:nth-child(1) {
  transform: translate(0, 0);
}
.hive-cell:nth-child(2) {
  transform: translate(-75%, -50%);
}
.hive-cell:nth-child(3) {
  transform: translate(0, -100%);
}
.hive-cell:nth-child(4) {
  transform: translate(75%, -50%);
}
.hive-cell:nth-child(5) {
  transform: translate(75%, 50%);
}
.hive-cell:nth-child(6) {
  transform: translate(0, 100%);
}
.hive-cell:nth-child(7) {
  transform: translate(-75%, 50%);
}
.hive-actions {
  margin-top: 1em;
  text-align: center;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  flex-wrap: nowrap;
}
.hive-actions button {
  transition: all 0.2s ease;
}
.hive-actions button:active {
  box-shadow: 0 5px $bl-grey;
  transform: translateY(4px);
}
polygon.cell-fill:active {
  transform: scale(0.9);
}
polygon.cell-fill {
  transition: all 1s;
  transform-origin: 50% 50%;
}
.hive-action {
  padding: 15px;
  flex: 1;
  background-color: #fff;
  font-size: 18px;
  margin: 0 12px;
  color: #333;
  border: 1px solid #ccc;
  border-radius: 40px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  cursor: pointer;
}
.hive-action__shuffle {
  background: url(../assets/shuffle.svg) center no-repeat;
  background-color: #fff;
  background-size: auto;
  background-size: 60%;
  height: 50px;
  min-width: 50px;
}
html.dark {
  .hive-action,
  .hive-action__shuffle {
    background-color: $bl-grey;
  }
  .user-guess {
    color: $bl-grey;
  }
}

@media only screen and (max-height: 650px) {
  .sb-controls {
    top: max(65%, 420px);
  }
  .hive-cell {
    top: 30%;
    left: 33%;
    width: 33%;
    height: 28%;
  }
  .hive-actions {
    margin-top: 0;
  }
}

@media (orientation: landscape) {
  .sb-controls {
    top: max(65%, 500px);
  }
  .hive-actions {
    padding-bottom: 2rem;
  }
}
</style>
