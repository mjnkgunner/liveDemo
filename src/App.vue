<script setup>
import { ref, onMounted } from 'vue'

// Constants and reactive state
const images = [
  '/QooBee1.jpg', '/QooBee2.jpg', '/QooBee3.jpg', '/QooBee4.jpg',
  '/QooBee5.jpg', '/QooBee6.jpg', '/QooBee7.jpg', '/QooBee8.jpg',
  '/QooBee9.jpg', '/QooBee10.jpg', '/QooBee11.jpg', '/QooBee12.jpg',
  '/QooBee13.jpg', '/QooBee14.jpg', '/QooBee15.jpg', '/QooBee16.jpg'
]

const gameCards = ref([])
const firstCard = ref(null)
const secondCard = ref(null)
const lockBoard = ref(false)
const matchedCards = ref(0)
const timeLeft = ref(180)
const message = ref('')
const timerInterval = ref(null)

// Methods
const startGame = () => {
  gameCards.value = shuffle([...images, ...images].map((img, index) => ({
    id: index,
    image: img,
    isFlipped: false,
    isMatched: false
  })))
  firstCard.value = secondCard.value = null
  matchedCards.value = 0
  lockBoard.value = false
  message.value = ''
  timeLeft.value = 180
  
  clearInterval(timerInterval.value)
  timerInterval.value = setInterval(countdown, 1000)
}

const flipCard = (card) => {
  if (lockBoard.value) return
  if (card.isFlipped) return

  card.isFlipped = true

  if (!firstCard.value) {
    firstCard.value = card
  } else {
    secondCard.value = card
    checkForMatch()
  }
}

const checkForMatch = () => {
  if (firstCard.value.image === secondCard.value.image) {
    firstCard.value.isMatched = true
    secondCard.value.isMatched = true
    matchedCards.value += 2

    if (matchedCards.value === gameCards.value.length) {
      message.value = 'Bạn đã chiến thắng!'
    }
    resetBoard()
  } else {
    lockBoard.value = true
    setTimeout(() => {
      firstCard.value.isFlipped = false
      secondCard.value.isFlipped = false
      resetBoard()
    }, 1000)
  }
}

const resetBoard = () => {
  firstCard.value = null
  secondCard.value = null
  lockBoard.value = false
}

const countdown = () => {
  if (timeLeft.value > 0) {
    timeLeft.value--
  } else {
    clearInterval(timerInterval.value)
    gameOver()
  }
}

const gameOver = () => {
  lockBoard.value = true
  message.value = 'Hết giờ!!!!!!'
}

const shuffle = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

// Lifecycle hooks
onMounted(() => {
  startGame()
})
</script>

<template>
  <div class="game-container">
    <div id="timer" :class="{ flash: timeLeft % 2 === 0 }">
      Thời gian còn lại: {{ Math.floor(timeLeft / 60) }}:{{ (timeLeft % 60).toString().padStart(2, '0') }}
    </div>
    <div id="message">{{ message }}</div>
    <button id="startGame" @click="startGame">Bắt đầu game</button>
    
    <div id="gameBoard">
      <div
        v-for="card in gameCards"
        :key="card.id"
        class="card"
        :class="{ flipped: card.isFlipped, matched: card.isMatched }"
        @click="flipCard(card)"
      >
        <div class="card-inner">
          <div class="card-front">
            <img :src="card.image" alt="card" />
          </div>
          <div class="card-back"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Copy all your existing CSS here, removing any jQuery-specific styles */
#gameBoard {
  width: 1340px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: repeat(8, 160px);
  grid-gap: 10px;
}

.card {
  width: 160px;
  height: 160px;
  perspective: 1000px;
  cursor: pointer;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front, .card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card-front {
  background-color: lightgray;
  transform: rotateY(180deg);
}

.card-front img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.card-back {
  background-image: url(./assets/images/bg.jpg);
  background-size: cover;
  background-repeat: no-repeat;
  border: 1px solid #f8852e;
}

.matched .card-inner {
  background-color: lightgreen;
  pointer-events: none;
}

#message {
  margin-top: 20px;
  font-size: 40px;
  color: red;
}

#startGame {
  padding: 20px 40px;
  font-size: 24px;
  cursor: pointer;
  outline: none;
  border: none;
  margin: 20px 20px;
  border-radius: 5px;
  background-color: #f36f24;
  color: white;
  font-weight: 600;
}

#timer {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  transition: color 0.5s ease, transform 0.2s ease;
}

#timer.flash {
  color: red;
  transform: scale(1);
}
</style>
