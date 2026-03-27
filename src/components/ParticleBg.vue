<template>
  <!-- 粒子背景画布 -->
  <canvas ref="canvasRef" class="particle-canvas"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// ========== Props 配置 ==========
const props = defineProps({
  // 粒子数量
  count: { type: Number, default: 80 },
  // 粒子颜色
  color: { type: String, default: 'rgba(120, 160, 255, 0.6)' },
  // 连线距离阈值
  linkDistance: { type: Number, default: 150 },
  // 连线颜色
  linkColor: { type: String, default: 'rgba(120, 160, 255, 0.15)' }
})

const canvasRef = ref(null)
let ctx = null
let particles = []
let animationId = null

// ========== 粒子类 ==========
class Particle {
  constructor(w, h) {
    this.x = Math.random() * w
    this.y = Math.random() * h
    this.vx = (Math.random() - 0.5) * 0.8
    this.vy = (Math.random() - 0.5) * 0.8
    this.radius = Math.random() * 2 + 1
    this.opacity = Math.random() * 0.5 + 0.3
  }

  // 更新粒子位置
  update(w, h) {
    this.x += this.vx
    this.y += this.vy
    // 边界反弹
    if (this.x < 0 || this.x > w) this.vx *= -1
    if (this.y < 0 || this.y > h) this.vy *= -1
  }

  // 绘制粒子
  draw(ctx, color) {
    ctx.beginPath()
    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2)
    ctx.fillStyle = color
    ctx.globalAlpha = this.opacity
    ctx.fill()
    ctx.globalAlpha = 1
  }
}

// ========== 初始化粒子 ==========
function initParticles(w, h) {
  particles = []
  for (let i = 0; i < props.count; i++) {
    particles.push(new Particle(w, h))
  }
}

// ========== 绘制连线 ==========
function drawLinks() {
  for (let i = 0; i < particles.length; i++) {
    for (let j = i + 1; j < particles.length; j++) {
      const dx = particles[i].x - particles[j].x
      const dy = particles[i].y - particles[j].y
      const dist = Math.sqrt(dx * dx + dy * dy)
      if (dist < props.linkDistance) {
        const opacity = 1 - dist / props.linkDistance
        ctx.beginPath()
        ctx.moveTo(particles[i].x, particles[i].y)
        ctx.lineTo(particles[j].x, particles[j].y)
        ctx.strokeStyle = props.linkColor
        ctx.globalAlpha = opacity * 0.5
        ctx.lineWidth = 0.5
        ctx.stroke()
        ctx.globalAlpha = 1
      }
    }
  }
}

// ========== 动画循环 ==========
function animate() {
  const canvas = canvasRef.value
  if (!canvas) return
  const w = canvas.width
  const h = canvas.height

  ctx.clearRect(0, 0, w, h)

  particles.forEach(p => {
    p.update(w, h)
    p.draw(ctx, props.color)
  })

  drawLinks()
  animationId = requestAnimationFrame(animate)
}

// ========== 处理窗口缩放 ==========
function handleResize() {
  const canvas = canvasRef.value
  if (!canvas) return
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
  initParticles(canvas.width, canvas.height)
}

onMounted(() => {
  const canvas = canvasRef.value
  ctx = canvas.getContext('2d')
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
  initParticles(canvas.width, canvas.height)
  animate()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.particle-canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}
</style>
