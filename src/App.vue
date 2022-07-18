<script setup lang="ts">
const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)

const WIDTH = 600
const HEIGHT = 600
interface Point {
  x: number
  y: number
}
interface Branch {
  start: Point
  length: number
  theta: number
}
function init() {
  ctx.strokeStyle = '#4f3916'
  step({
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    theta: -Math.PI / 2,
  })
}

const pendingTasks: Function[] = []

function step(b: Branch, depth = 0) {
  const { theta, length, start } = b
  const { x, y } = start
  const end = getEndPoint(b)
  drawBranch(b)
  if (x < 0 || x > WIDTH || y < 0 || y > HEIGHT || depth > 18)
    return
  if (depth < 4 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: length + (Math.random() * 10 - 5),
      theta: theta - 0.3 * Math.random(),
    }, depth + 1))
  }
  if (depth < 4 || Math.random() < 0.5) {
    pendingTasks.push(() => step({
      start: end,
      length: length + (Math.random() * 10 - 5),
      theta: theta + 0.3 * Math.random(),
    }, depth + 1))
  }
}

function frame() {
  const task = [...pendingTasks]
  pendingTasks.length = 0
  task.forEach(fn => fn())
}

let frameCount = 0

function startFrame() {
  requestAnimationFrame(() => {
    frameCount += 1
    if (frameCount % 10 === 0)
      frame()

    startFrame()
  })
}

startFrame()

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}

function getEndPoint(b: Branch) {
  const { start, theta, length } = b
  return {
    x: start.x + length * Math.cos(theta),
    y: start.y + length * Math.sin(theta),
  }
}
function drawBranch(b: Branch) {
  const { start } = b

  lineTo(start, getEndPoint(b))
}
onMounted(() => {
  init()
})
</script>

<template>
  <canvas ref="el" :width="WIDTH" :height="HEIGHT" border />
</template>
