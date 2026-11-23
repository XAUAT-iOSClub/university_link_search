<template>
  <div class="account-card">
    <template v-if="isLogin">
      <div class="account-avatar account-avatar-active">
        <Icon class="text-[26px]">
          <AccountCircleRound/>
        </Icon>
      </div>
      <p class="mt-3 text-[17px] font-semibold text-[var(--ink)]">{{ username }}</p>
      <p class="mt-1 text-[12.5px] text-[var(--ink-2)]">点击进入账号管理</p>
    </template>

    <template v-else>
      <div class="account-avatar">
        <Icon class="text-[26px]">
          <AccountCircleRound/>
        </Icon>
      </div>
      <p class="mt-3 text-[17px] font-semibold text-[var(--ink)]">您尚未登录</p>
      <p class="mt-1 text-[12.5px] leading-relaxed text-[var(--ink-2)]">
        登录后可云端同步个性化设置、网站捷径和便笺
      </p>
      <p class="mt-3 text-[11.5px] text-[var(--accent)]">点击图标登录</p>
    </template>
  </div>
</template>

<script setup lang="ts">
import {AccountCircleRound} from '@vicons/material'
import {Icon} from '@vicons/utils'
import {onMounted, ref} from 'vue'
import { useAuthorizationStore } from '../stores/Authorization'

const store = useAuthorizationStore()
const isLogin = ref(store.isAuthenticated);
const username = ref<string>('')

onMounted(() => {
  if (isLogin.value) {
    const userInfo = store.getClubData
    if (userInfo) {
      username.value = userInfo.name
    }
  }
})
</script>

<style scoped>
.account-card {
  width: 300px;
  padding: 22px;
  border-radius: 22px;
  border: 1px solid var(--glass-border);
  background: var(--glass-strong);
  box-shadow: 0 24px 64px rgba(15, 23, 42, 0.18);
  -webkit-backdrop-filter: blur(30px) saturate(180%);
  backdrop-filter: blur(30px) saturate(180%);
  text-align: center;
}

.dark .account-card {
  box-shadow: 0 24px 64px rgba(0, 0, 0, 0.7);
}

.account-avatar {
  display: flex;
  width: 52px;
  height: 52px;
  margin: 0 auto;
  align-items: center;
  justify-content: center;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.05);
  color: var(--ink-2);
}

.dark .account-avatar {
  background: rgba(255, 255, 255, 0.08);
}

.account-avatar-active {
  background: var(--accent-soft);
  color: var(--accent);
}
</style>
