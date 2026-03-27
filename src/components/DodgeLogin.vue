<template>
  <div class="dodge-login-wrapper">
    <!-- 粒子背景 -->
    <ParticleBg />

    <!-- 登录卡片 -->
    <div class="login-card" ref="cardRef">
      <!-- Logo 和标题 -->
      <div class="login-header">
        <div class="logo-circle">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
            <path d="M15 3h4a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2h-4"/>
            <polyline points="10 17 15 12 10 7"/>
            <line x1="15" y1="12" x2="3" y2="12"/>
          </svg>
        </div>
        <h1 class="login-title">{{ title }}</h1>
        <p class="login-subtitle">{{ subtitle }}</p>
      </div>

      <!-- 表单区域 -->
      <div class="login-form">
        <!-- 账号输入 -->
        <div class="input-group" :class="{ focused: usernameFocused, filled: username }">
          <div class="input-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/>
              <circle cx="12" cy="7" r="4"/>
            </svg>
          </div>
          <input
            id="dodge-username"
            type="text"
            v-model="username"
            :placeholder="usernameLabel"
            @focus="usernameFocused = true"
            @blur="usernameFocused = false"
            autocomplete="username"
          />
          <div class="input-glow"></div>
        </div>

        <!-- 密码输入 -->
        <div class="input-group" :class="{ focused: passwordFocused, filled: password }">
          <div class="input-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <rect x="3" y="11" width="18" height="11" rx="2" ry="2"/>
              <path d="M7 11V7a5 5 0 0 1 10 0v4"/>
            </svg>
          </div>
          <input
            id="dodge-password"
            :type="showPassword ? 'text' : 'password'"
            v-model="password"
            :placeholder="passwordLabel"
            @focus="passwordFocused = true"
            @blur="passwordFocused = false"
            @keyup.enter="handleLogin"
            autocomplete="current-password"
          />
          <!-- 密码可见切换 -->
          <button class="toggle-password" @click="showPassword = !showPassword" type="button" tabindex="-1">
            <svg v-if="!showPassword" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
              <circle cx="12" cy="12" r="3"/>
            </svg>
            <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/>
              <line x1="1" y1="1" x2="23" y2="23"/>
            </svg>
          </button>
          <div class="input-glow"></div>
        </div>

        <!-- 登录按钮容器 - 相对定位用于按钮躲闪 -->
        <div
          class="btn-container"
          ref="btnContainerRef"
          @mousemove="handleMouseMove"
          @mouseleave="handleMouseLeave"
        >
          <button
            id="dodge-login-btn"
            class="login-btn"
            :class="{
              dodging: !canLogin,
              ready: canLogin,
              shaking: isShaking
            }"
            :style="buttonStyle"
            @click="handleLogin"
            :disabled="!canLogin"
          >
            <span class="btn-text">{{ canLogin ? '立即登录' : '请先填写信息' }}</span>
            <span class="btn-shine"></span>
          </button>
        </div>

        <!-- 提示信息 -->
        <transition name="hint-fade">
          <p class="hint-text" v-if="!canLogin && showHint">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="hint-icon">
              <circle cx="12" cy="12" r="10"/>
              <line x1="12" y1="16" x2="12" y2="12"/>
              <line x1="12" y1="8" x2="12.01" y2="8"/>
            </svg>
            {{ hintMessage }}
          </p>
        </transition>
      </div>

      <!-- 底部信息 -->
      <div class="login-footer" v-if="showFooter">
        <a href="javascript:void(0)" class="footer-link" @click="$emit('forgotPassword')">忘记密码？</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import ParticleBg from './ParticleBg.vue'

// ========== Props 配置（方便复用） ==========
const props = defineProps({
  // 标题
  title: { type: String, default: '用户登录' },
  // 副标题
  subtitle: { type: String, default: '欢迎回来，请登录您的账号' },
  // 账号输入框占位符
  usernameLabel: { type: String, default: '请输入账号' },
  // 密码输入框占位符
  passwordLabel: { type: String, default: '请输入密码' },
  // 躲闪触发距离（鼠标距离按钮多近时开始躲闪）
  dodgeDistance: { type: Number, default: 120 },
  // 躲闪移动距离
  dodgeMoveDistance: { type: Number, default: 100 },
  // 躲闪动画过渡时间（秒）
  dodgeSpeed: { type: Number, default: 0.25 },
  // 是否显示底部链接
  showFooter: { type: Boolean, default: true }
})

// ========== 事件定义 ==========
const emit = defineEmits(['login', 'forgotPassword'])

// ========== 响应式状态 ==========
const username = ref('')
const password = ref('')
const showPassword = ref(false)
const usernameFocused = ref(false)
const passwordFocused = ref(false)
const showHint = ref(false)
const isShaking = ref(false)

// 按钮位移
const btnOffsetX = ref(0)
const btnOffsetY = ref(0)

// 引用
const cardRef = ref(null)
const btnContainerRef = ref(null)

// ========== 计算属性 ==========

// 是否可以登录（账号密码都已填写）
const canLogin = computed(() => {
  return username.value.trim() !== '' && password.value.trim() !== ''
})

// 提示信息
const hintMessage = computed(() => {
  if (!username.value.trim() && !password.value.trim()) return '请输入账号和密码'
  if (!username.value.trim()) return '请输入账号'
  if (!password.value.trim()) return '请输入密码'
  return ''
})

// 按钮样式
const buttonStyle = computed(() => {
  if (canLogin.value) {
    return {
      transform: 'translate(0, 0)',
      transition: `transform ${props.dodgeSpeed}s cubic-bezier(0.34, 1.56, 0.64, 1)`
    }
  }
  return {
    transform: `translate(${btnOffsetX.value}px, ${btnOffsetY.value}px)`,
    transition: `transform ${props.dodgeSpeed}s cubic-bezier(0.34, 1.56, 0.64, 1)`
  }
})

// ========== 当填写完成时重置按钮位置 ==========
watch(canLogin, (val) => {
  if (val) {
    btnOffsetX.value = 0
    btnOffsetY.value = 0
    showHint.value = false
  }
})

// ========== 鼠标移动 ==========
function handleMouseMove(e) {
  // 如果可以登录，不躲闪
  if (canLogin.value) return

  showHint.value = true

  const container = btnContainerRef.value
  if (!container) return

  const containerRect = container.getBoundingClientRect()

  // 获取按钮元素
  const btn = container.querySelector('.login-btn')
  if (!btn) return

  const btnRect = btn.getBoundingClientRect()

  // 按钮中心坐标（相对于视口）
  const btnCenterX = btnRect.left + btnRect.width / 2
  const btnCenterY = btnRect.top + btnRect.height / 2

  // 鼠标坐标
  const mouseX = e.clientX
  const mouseY = e.clientY

  // 计算鼠标到按钮中心的距离
  const dx = mouseX - btnCenterX
  const dy = mouseY - btnCenterY
  const distance = Math.sqrt(dx * dx + dy * dy)

  // 如果鼠标足够近，躲闪
  if (distance < props.dodgeDistance) {
    // 计算躲闪方向（远离鼠标）
    const angle = Math.atan2(dy, dx)

    // 计算躲闪力度（鼠标越近，躲闪越远）
    const force = (1 - distance / props.dodgeDistance) * props.dodgeMoveDistance

    // 新的偏移量
    let newOffsetX = -Math.cos(angle) * force + (Math.random() - 0.5) * 20
    let newOffsetY = -Math.sin(angle) * force + (Math.random() - 0.5) * 20

    // 边界限制：确保按钮不会飞出容器
    const maxOffsetX = (containerRect.width - btnRect.width) / 2
    const maxOffsetY = (containerRect.height - btnRect.height) / 2

    newOffsetX = Math.max(-maxOffsetX, Math.min(maxOffsetX, newOffsetX))
    newOffsetY = Math.max(-maxOffsetY, Math.min(maxOffsetY, newOffsetY))

    btnOffsetX.value = newOffsetX
    btnOffsetY.value = newOffsetY

    // 触发抖动效果
    isShaking.value = true
    setTimeout(() => { isShaking.value = false }, 300)
  }
}

// ========== 鼠标离开容器 ==========
function handleMouseLeave() {
  if (!canLogin.value) {
    // 鼠标离开后按钮缓慢回到中心
    btnOffsetX.value = 0
    btnOffsetY.value = 0
  }
}

// ========== 登录处理 ==========
function handleLogin() {
  if (!canLogin.value) return
  emit('login', {
    username: username.value.trim(),
    password: password.value.trim()
  })
}
</script>

<style scoped>
/* ========== 整体包裹层 ========== */
.dodge-login-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #0a0e27 0%, #151b3d 30%, #1a1040 60%, #0d1130 100%);
  position: relative;
  overflow: hidden;
}

/* 背景装饰光圈 */
.dodge-login-wrapper::before {
  content: '';
  position: absolute;
  width: 600px;
  height: 600px;
  background: radial-gradient(circle, rgba(99, 102, 241, 0.15) 0%, transparent 70%);
  top: -200px;
  right: -100px;
  animation: float 8s ease-in-out infinite;
}

.dodge-login-wrapper::after {
  content: '';
  position: absolute;
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, rgba(168, 85, 247, 0.12) 0%, transparent 70%);
  bottom: -150px;
  left: -100px;
  animation: float 10s ease-in-out infinite reverse;
}

@keyframes float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-30px) rotate(5deg); }
}

/* ========== 登录卡片 ========== */
.login-card {
  position: relative;
  z-index: 10;
  width: 420px;
  padding: 48px 40px;
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow:
    0 32px 64px rgba(0, 0, 0, 0.4),
    0 0 0 1px rgba(255, 255, 255, 0.05) inset,
    0 0 80px rgba(99, 102, 241, 0.06);
  animation: cardAppear 0.8s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes cardAppear {
  0% {
    opacity: 0;
    transform: translateY(40px) scale(0.95);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* ========== 头部区域 ========== */
.login-header {
  text-align: center;
  margin-bottom: 40px;
}

.logo-circle {
  width: 64px;
  height: 64px;
  margin: 0 auto 20px;
  background: linear-gradient(135deg, #6366f1, #8b5cf6, #a855f7);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 32px rgba(99, 102, 241, 0.35);
  animation: logoGlow 3s ease-in-out infinite;
}

@keyframes logoGlow {
  0%, 100% { box-shadow: 0 8px 32px rgba(99, 102, 241, 0.35); }
  50% { box-shadow: 0 8px 48px rgba(99, 102, 241, 0.55); }
}

.logo-circle svg {
  width: 28px;
  height: 28px;
  color: white;
}

.login-title {
  font-size: 26px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 1px;
  margin-bottom: 8px;
}

.login-subtitle {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.45);
  font-weight: 300;
}

/* ========== 输入框组 ========== */
.input-group {
  position: relative;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  transition: all 0.3s ease;
  overflow: hidden;
}

.input-group:hover {
  border-color: rgba(99, 102, 241, 0.3);
  background: rgba(255, 255, 255, 0.06);
}

.input-group.focused {
  border-color: rgba(99, 102, 241, 0.6);
  background: rgba(255, 255, 255, 0.06);
  box-shadow: 0 0 24px rgba(99, 102, 241, 0.12);
}

.input-group.filled {
  border-color: rgba(99, 102, 241, 0.3);
}

.input-icon {
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.input-icon svg {
  width: 18px;
  height: 18px;
  color: rgba(255, 255, 255, 0.35);
  transition: color 0.3s ease;
}

.input-group.focused .input-icon svg,
.input-group.filled .input-icon svg {
  color: #8b5cf6;
}

.input-group input {
  flex: 1;
  height: 52px;
  background: transparent;
  border: none;
  outline: none;
  color: #fff;
  font-size: 15px;
  font-family: inherit;
  padding-right: 16px;
  letter-spacing: 0.3px;
}

.input-group input::placeholder {
  color: rgba(255, 255, 255, 0.25);
  font-weight: 300;
}

/* 输入框聚焦时底部发光线 */
.input-glow {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%) scaleX(0);
  width: 80%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #6366f1, #a855f7, transparent);
  transition: transform 0.4s ease;
}

.input-group.focused .input-glow {
  transform: translateX(-50%) scaleX(1);
}

/* 密码显示切换按钮 */
.toggle-password {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  cursor: pointer;
  margin-right: 8px;
  border-radius: 10px;
  transition: background 0.2s ease;
}

.toggle-password:hover {
  background: rgba(255, 255, 255, 0.06);
}

.toggle-password svg {
  width: 18px;
  height: 18px;
  color: rgba(255, 255, 255, 0.3);
  transition: color 0.2s ease;
}

.toggle-password:hover svg {
  color: rgba(255, 255, 255, 0.6);
}

/* ========== 按钮容器（躲闪活动区域） ========== */
.btn-container {
  position: relative;
  width: 100%;
  height: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 8px;
}

/* ========== 登录按钮 ========== */
.login-btn {
  position: relative;
  width: 100%;
  max-width: 280px;
  height: 52px;
  border: none;
  border-radius: 14px;
  font-size: 16px;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  overflow: hidden;
  transition: all 0.3s ease;
  letter-spacing: 1px;
  user-select: none;
}

/* 躲闪状态的按钮 */
.login-btn.dodging {
  background: linear-gradient(135deg, #374151, #4b5563);
  color: rgba(255, 255, 255, 0.6);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
  cursor: not-allowed;
}

.login-btn.dodging:hover {
  /* 被hover时不改变样式，因为会躲开 */
}

/* 抖动动画 */
.login-btn.shaking {
  animation: buttonShake 0.3s ease;
}

@keyframes buttonShake {
  0%, 100% { rotate: 0deg; }
  25% { rotate: -3deg; }
  75% { rotate: 3deg; }
}

/* 就绪状态的按钮 */
.login-btn.ready {
  background: linear-gradient(135deg, #6366f1, #8b5cf6, #a855f7);
  color: #fff;
  box-shadow: 0 8px 32px rgba(99, 102, 241, 0.4);
  cursor: pointer;
  animation: btnReady 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes btnReady {
  0% { transform: scale(0.9); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

.login-btn.ready:hover {
  box-shadow: 0 12px 40px rgba(99, 102, 241, 0.55);
  transform: translateY(-2px) !important;
}

.login-btn.ready:active {
  transform: scale(0.97) !important;
  box-shadow: 0 4px 16px rgba(99, 102, 241, 0.3);
}

/* 按钮文字 */
.btn-text {
  position: relative;
  z-index: 2;
}

/* 按钮光泽效果 */
.btn-shine {
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.15), transparent);
  z-index: 1;
}

.login-btn.ready .btn-shine {
  animation: shine 3s ease-in-out infinite;
}

@keyframes shine {
  0% { left: -100%; }
  50% { left: 100%; }
  100% { left: 100%; }
}

/* ========== 提示信息 ========== */
.hint-text {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  font-size: 13px;
  color: rgba(251, 191, 36, 0.8);
  text-align: center;
  margin-top: 4px;
}

.hint-icon {
  width: 14px;
  height: 14px;
  flex-shrink: 0;
}

/* 提示淡入淡出动画 */
.hint-fade-enter-active,
.hint-fade-leave-active {
  transition: all 0.3s ease;
}
.hint-fade-enter-from,
.hint-fade-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

/* ========== 底部链接 ========== */
.login-footer {
  text-align: center;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
}

.footer-link {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.4);
  text-decoration: none;
  transition: color 0.2s ease;
}

.footer-link:hover {
  color: #8b5cf6;
}

/* ========== 响应式适配 ========== */
@media (max-width: 480px) {
  .login-card {
    width: calc(100% - 32px);
    padding: 36px 24px;
    border-radius: 20px;
  }
  .login-title {
    font-size: 22px;
  }
  .btn-container {
    height: 100px;
  }
}
</style>
