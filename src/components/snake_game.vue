<template>
  <div class="game-container">
    <div class="game-header">
      <div class="score">分数: {{ score }}</div>
      <div class="high-score">最高分: {{ highScore }}</div>
    </div>

    <div class="game-board">
      <div
        v-for="(cell, index) in flatBoard"
        :key="index"
        :class="['cell', getCellClass(cell)]"
      >
        <template
          v-if="isSnakeHead(index % BOARD_SIZE, Math.floor(index / BOARD_SIZE))"
        >
          <div class="snake-head" :class="getHeadDirection()">
            <div class="face">
              <div class="eyes">
                <div class="eye">
                  <div class="pupil"></div>
                </div>
                <div class="eye">
                  <div class="pupil"></div>
                </div>
              </div>
              <div class="mouth" :class="{ 'mouth-open': isEating }">
                <div class="tongue" v-if="isEating"></div>
              </div>
            </div>
          </div>
        </template>
        <div v-if="cell === 'food'" class="food">🌰</div>
      </div>
    </div>

    <div class="controls">
      <button @click="startGame" :disabled="isPlaying">开始游戏</button>
      <button @click="pauseGame" :disabled="!isPlaying">暂停</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

const BOARD_SIZE = 20
const INITIAL_SPEED = 150
const board = ref<string[][]>([])
const snake = ref<{ x: number; y: number }[]>([])
const direction = ref<string>('right')
const food = ref<{ x: number; y: number }>({ x: 0, y: 0 })
const score = ref(0)
const highScore = ref(0)
const isPlaying = ref(false)
const isEating = ref(false)
const gameInterval = ref<ReturnType<typeof setInterval> | null>(null)

// 添加计算属性来扁平化游戏板
const flatBoard = computed(() => board.value.flat())

// 初始化游戏板
const initBoard = () => {
  board.value = Array(BOARD_SIZE)
    .fill(null)
    .map(() => Array(BOARD_SIZE).fill('empty'))
}

// 初始化蛇的位置
const initSnake = () => {
  snake.value = [
    { x: 5, y: 5 },
    { x: 4, y: 5 },
    { x: 3, y: 5 },
  ]
  direction.value = 'right'
}

// 生成食物
const generateFood = () => {
  let x: number, y: number
  do {
    x = Math.floor(Math.random() * BOARD_SIZE)
    y = Math.floor(Math.random() * BOARD_SIZE)
  } while (snake.value.some(segment => segment.x === x && segment.y === y))

  food.value = { x, y }
}

// 更新游戏板
const updateBoard = () => {
  initBoard()
  snake.value.forEach((segment, index) => {
    board.value[segment.y][segment.x] = index === 0 ? 'head' : 'body'
  })
  board.value[food.value.y][food.value.x] = 'food'
}

// 移动蛇
const moveSnake = () => {
  const head = { ...snake.value[0] }

  switch (direction.value) {
    case 'up':
      head.y--
      break
    case 'down':
      head.y++
      break
    case 'left':
      head.x--
      break
    case 'right':
      head.x++
      break
  }

  // 检查碰撞
  if (checkCollision(head)) {
    gameOver()
    return
  }

  snake.value.unshift(head)

  // 检查是否吃到食物
  if (head.x === food.value.x && head.y === food.value.y) {
    score.value += 10
    isEating.value = true
    setTimeout(() => (isEating.value = false), 300)
    generateFood()
  } else {
    snake.value.pop()
  }

  updateBoard()
}

// 检查碰撞
const checkCollision = (head: { x: number; y: number }) => {
  return (
    head.x < 0 ||
    head.x >= BOARD_SIZE ||
    head.y < 0 ||
    head.y >= BOARD_SIZE ||
    snake.value.some(segment => segment.x === head.x && segment.y === head.y)
  )
}

// 开始游戏
const startGame = () => {
  initBoard()
  initSnake()
  generateFood()
  isPlaying.value = true
  gameInterval.value = setInterval(moveSnake, INITIAL_SPEED)
}

// 暂停游戏
const pauseGame = () => {
  if (gameInterval.value) {
    clearInterval(gameInterval.value)
    gameInterval.value = null
  }
  isPlaying.value = false
}

// 游戏结束
const gameOver = () => {
  pauseGame()
  if (score.value > highScore.value) {
    highScore.value = score.value
  }
  score.value = 0
}

// 键盘控制
const handleKeydown = (e: KeyboardEvent) => {
  if (!isPlaying.value) return

  const key = e.key.toLowerCase()
  const opposites = {
    up: 'down',
    down: 'up',
    left: 'right',
    right: 'left',
  }

  if (['arrowup', 'w'].includes(key) && direction.value !== opposites.up) {
    direction.value = 'up'
  } else if (
    ['arrowdown', 's'].includes(key) &&
    direction.value !== opposites.down
  ) {
    direction.value = 'down'
  } else if (
    ['arrowleft', 'a'].includes(key) &&
    direction.value !== opposites.left
  ) {
    direction.value = 'left'
  } else if (
    ['arrowright', 'd'].includes(key) &&
    direction.value !== opposites.right
  ) {
    direction.value = 'right'
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  initBoard()
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  if (gameInterval.value) {
    clearInterval(gameInterval.value)
  }
})

// 修改 isSnakeHead 函数提高性能
const isSnakeHead = (x: number, y: number) => {
  const head = snake.value[0]
  return head && head.x === x && head.y === y
}

const getCellClass = (cell: string) => ({
  'snake-head': cell === 'head',
  'snake-body': cell === 'body',
  food: cell === 'food',
})

// 添加获取蛇头方向的方法
const getHeadDirection = () => ({
  'head-up': direction.value === 'up',
  'head-down': direction.value === 'down',
  'head-left': direction.value === 'left',
  'head-right': direction.value === 'right',
})
</script>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 30px;
  background: linear-gradient(135deg, #f0f8ff 0%, #e6f3ff 100%);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.game-header {
  display: flex;
  gap: 20px;
  font-size: 1.2em;
  font-weight: bold;
  color: #4a5568;
}

.game-board {
  border: 4px solid #e2e8f0;
  border-radius: 10px;
  overflow: hidden;
  background: #fff;
  width: 500px;
  height: 500px;
  display: grid;
  grid-template-columns: repeat(20, 1fr);
  grid-template-rows: repeat(20, 1fr);
  gap: 0;
}

.cell {
  width: 100%;
  height: 100%;
  border: 1px solid #f0f4f8;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.snake-head {
  width: 100%;
  height: 100%;
  background: #48bb78;
  border-radius: 12px;
  position: relative;
  transition: transform 0.1s linear;
  will-change: transform;
}

.snake-body {
  width: 100%;
  height: 100%;
  background: #68d391;
  border-radius: 6px;
  transform: scale(0.85);
  transition: all 0.1s linear;
}

.food {
  font-size: 15px;
  animation: gentleBounce 1s infinite alternate;
  transform-origin: center;
  will-change: transform;
}

.eyes {
  position: absolute;
  top: 20%;
  width: 100%;
  display: flex;
  justify-content: space-around;
  padding: 0 15%;
}

.eye {
  width: 8px;
  height: 8px;
  background: white;
  border-radius: 50%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.pupil {
  width: 4px;
  height: 4px;
  background: black;
  border-radius: 50%;
}

.mouth {
  position: absolute;
  bottom: 20%;
  left: 50%;
  transform: translateX(-50%);
  width: 10px;
  height: 5px;
  background: #e53e3e;
  border-radius: 0 0 5px 5px;
  transition: all 0.2s;
}

.mouth-open {
  height: 10px;
  border-radius: 50%;
}

.tongue {
  position: absolute;
  bottom: -5px;
  left: 50%;
  transform: translateX(-50%);
  width: 4px;
  height: 8px;
  background: #fc8181;
  border-radius: 2px;
  animation: tongue-flick 0.3s infinite;
}

.controls {
  display: flex;
  gap: 10px;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  background: #4299e1;
  color: white;
  cursor: pointer;
  transition: all 0.2s;
}

button:hover:not(:disabled) {
  background: #3182ce;
  transform: translateY(-2px);
}

button:disabled {
  background: #cbd5e0;
  cursor: not-allowed;
}

@keyframes wiggle {
  0% {
    transform: rotate(-5deg);
  }
  50% {
    transform: rotate(5deg);
  }
  100% {
    transform: rotate(-5deg);
  }
}

@keyframes gentleBounce {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(-1px);
  }
}

/* 添加操作说明样式 */
.instructions {
  margin-bottom: 20px;
  text-align: center;
  background: white;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.instructions h3 {
  margin: 0 0 10px 0;
  color: #4a5568;
}

.control-keys {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
}

.key-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.key-row {
  display: flex;
  gap: 5px;
}

.key {
  width: 30px;
  height: 30px;
  background: #edf2f7;
  border: 1px solid #cbd5e0;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: #4a5568;
  box-shadow: 0 2px 0 #cbd5e0;
}

/* 蛇头方向样式 */
.head-up {
  transform: rotate(-90deg);
}
.head-down {
  transform: rotate(90deg);
}
.head-left {
  transform: rotate(180deg);
}
.head-right {
  transform: rotate(0deg);
}

@keyframes tongue-flick {
  0% {
    height: 8px;
  }
  50% {
    height: 12px;
  }
  100% {
    height: 8px;
  }
}
</style>
