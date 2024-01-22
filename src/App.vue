<script setup lang="ts">
import GameHeader from "@/components/GameHeader.vue";
import GameFigure from "@/components/GameFigure.vue";
import GameWrongLeatters from "@/components/GameWrongLeatters.vue";
import GameWord from "@/components/GameWord.vue";
import GamePopup from "@/components/GamePopup.vue";
import GameNotification from "@/components/GameNotification.vue";
import {computed, ref, watch} from "vue";
import axios from "axios";

const word = ref('');
const getRandomWord = async () => {
  try {
    const {data} = await axios<{
      FirstName: string
    }>('https://api.randomdatatools.ru/?unescaped=false&params=FirstName')
    word.value = data.FirstName.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ''
  }
}

getRandomWord()

const letters = ref<string[]>([])
const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every(x => correctLetters.value.includes(x)))
const correctLetters = computed(() => letters.value.filter(
    x => word.value.includes(x)
))

const wrongLetters = computed(() => letters.value.filter(
    x => !word.value.includes(x)
))

const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch(correctLetters, () => {

  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({key}) => {
  if (isWin.value || isLose.value) {
    return;
  }
  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }

  if (/[а-яА-ЯёЁ]/.test(key)) {
    // console.log(key)
    letters.value.push(key.toLowerCase())
  }
});

const restart = async () => {
  await getRandomWord()
  letters.value = []
  popup.value?.close()
}
</script>
<template>
<!--  {{ word }}-->
  <GameHeader/>
  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length"/>

    <GameWrongLeatters :wrong-letters="wrongLetters"/>

    <GameWord :word="word" :correct-letters="correctLetters"/>
  </div>

  <!-- Container for final message -->
  <GamePopup ref="popup" :word="word" @restart="restart"/>

  <!-- Notification -->
  <GameNotification ref="notification"/>
</template>
