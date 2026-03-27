<template>
  <DodgeLogin
    title="用户登录"
    subtitle="欢迎回来，请登录您的账号"
    :dodgeDistance="130"
    :dodgeMoveDistance="110"
    @login="onLogin"
    @forgot-password="onForgotPassword"
  />
</template>

<script setup>
import { DodgeLogin } from './components'

// 登录回调
function onLogin({ username, password }) {
  alert(`登录成功！\n账号: ${username}\n密码: ${password}`)
  console.log('登录信息:', { username, password })
}

// 忘记密码回调
function onForgotPassword() {
  alert('跳转到找回密码页面...')
}
</script>

<style>
#app {
  width: 100%;
  height: 100vh;
}
</style>
