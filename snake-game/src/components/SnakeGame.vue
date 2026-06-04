<template>
  <div class="game-root">
    <h2>Snake Game 🐍</h2>
    <div class="info">
      <div>Score: {{ score }}</div>
      <div v-if="gameOver" class="game-over">Game Over — Press Enter to Restart</div>
    </div>
    <canvas ref="canvas" :width="width" :height="height" tabindex="0"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const canvas = ref(null)
const width = 400   
const height = 400
const scale = 20
const cols = width / scale
const rows = height / scale

let intervalId = null

const snake = ref([])
const dir = ref({ x: 1, y: 0 })
const food = ref({ x: 0, y: 0 })
const score = ref(0)
const gameOver = ref(false)

function reset() {
  snake.value = [{ x: Math.floor(cols / 2), y: Math.floor(rows / 2) }]
  dir.value = { x: 1, y:1 }
  placeFood()
  score.value = 0
  gameOver.value = false
}

function placeFood() {
  let x, y, coll
  do {
    x = Math.floor(Math.random() * cols)
    y = Math.floor(Math.random() * rows)
    coll = snake.value.some(s => s.x === x && s.y === y)
  } while (coll)
  food.value = { x, y }
}

function step() {
  if (gameOver.value) return
  const head = { x: snake.value[0].x + dir.value.x, y: snake.value[0].y + dir.value.y }

  // wrap around edges
  if (head.x < 0) head.x = cols - 1
  if (head.x >= cols) head.x = 0
  if (head.y < 0) head.y = rows - 1
  if (head.y >= rows) head.y = 0

  // self collision
  if (snake.value.some(s => s.x === head.x && s.y === head.y)) {
    gameOver.value = true
    return
  }

  snake.value.unshift(head)

  // food
  if (head.x === food.value.x && head.y === food.value.y) {
    score.value++
    placeFood()
  } else {
    snake.value.pop()
  }
}

function draw(ctx) {
  ctx.fillStyle = '#111'
  ctx.fillRect(0, 0, width, height)

  // draw food
  ctx.fillStyle = 'red'
  ctx.fillRect(food.value.x * scale, food.value.y * scale, scale, scale)

  // draw snake
  ctx.fillStyle = '#4caf50'
  for (const s of snake.value) {
    ctx.fillRect(s.x * scale, s.y * scale, scale - 1, scale - 1)
  }
}

function loop() {
  step()
  const ctx = canvas.value.getContext('2d')
  draw(ctx)
}

function handleKey(e) {
  const k = e.key
  if (gameOver.value && k === 'Enter') {
    reset()
    return
  }
  // prevent reversing
  if (k === 'ArrowUp' && dir.value.y !== 1) dir.value = { x: 0, y: -1 }
  if (k === 'ArrowDown' && dir.value.y !== -1) dir.value = { x: 0, y: 1 }
  if (k === 'ArrowLeft' && dir.value.x !== 1) dir.value = { x: -1, y: 0 }
  if (k === 'ArrowRight' && dir.value.x !== -1) dir.value = { x: 1, y: 0 }
}

onMounted(() => {
  reset()
  const el = canvas.value
  el.focus()
  el.addEventListener('keydown', handleKey)
  intervalId = setInterval(loop, 120)
})

onBeforeUnmount(() => {
  if (intervalId) clearInterval(intervalId)
  if (canvas.value) canvas.value.removeEventListener('keydown', handleKey)
})
</script>

<style scoped>
.game-root {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}
canvas {
  outline: none;
  border: 4px solid #222;
  background: #111;
}
.info {
  width: 400px;
  display: flex;
  justify-content: space-between;
  color: #fff;
}
.game-over {
  color: #ff5252;
}
h2 { color: #fff }
</style>
