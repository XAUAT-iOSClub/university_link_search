<template>
  <Teleport to="body">
    <div v-if="show" class="sheet-backdrop" @click="closeModal">
      <div class="sheet-panel" @click.stop>
        <div class="flex items-start justify-between gap-4">
          <div>
            <h2 class="text-[22px] font-semibold tracking-[-0.02em] text-[var(--ink)]">
              {{ isLogin ? '账号设置' : '登录' }}
            </h2>
            <p class="mt-1 text-[13px] text-[var(--ink-2)]">
              {{ isLogin ? '管理教务数据同步' : '使用 iMember 账户授权登录' }}
            </p>
          </div>
          <button class="close-btn" aria-label="关闭" @click="closeModal">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round">
              <path d="M6 6l12 12M18 6L6 18" />
            </svg>
          </button>
        </div>

        <template v-if="isLogin">
          <div class="mt-5 divide-y divide-[var(--hairline)]">
            <div class="setting-row">
              <div>
                <p class="setting-label">显示教务数据</p>
                <p class="setting-hint">在首页展示教务相关数据</p>
              </div>
              <n-switch v-model:value="isShow" @update:value="handleChange"/>
            </div>

            <template v-if="isShow">
              <div class="setting-row">
                <div>
                  <p class="setting-label">教务系统密码</p>
                  <p class="setting-hint">仅保存在本地</p>
                </div>
                <n-input
                  type="password"
                  show-password-on="mousedown"
                  placeholder="密码"
                  autosize
                  class="password-input"
                  v-model:value="eduPassword"
                />
              </div>

              <div class="pt-4">
                <n-button block round type="primary" @click="handleGetData">
                  获取相关数据
                </n-button>
              </div>
            </template>
          </div>
        </template>

        <template v-else>
          <div class="mt-6 rounded-[20px] border border-[var(--hairline)] bg-white/40 p-6 text-center dark:bg-white/[0.04]">
            <span class="mx-auto flex h-14 w-14 items-center justify-center rounded-full bg-[var(--accent-soft)] text-[var(--accent)]">
              <Icon icon="solar:user-id-bold" class="h-7 w-7" />
            </span>
            <p class="mt-4 text-[14px] font-medium text-[var(--ink)]">统一身份认证</p>
            <p class="mt-1 text-[12.5px] leading-relaxed text-[var(--ink-2)]">
              登录后可云端同步个性化设置、网站捷径和便笺
            </p>
          </div>

          <button type="button" class="primary-btn" @click="handleOAuthLogin">
            授权登录
          </button>

          <p class="mt-4 text-center text-[12px] leading-relaxed text-[var(--ink-2)]">
            点击授权登录后，将跳转到统一身份认证系统进行授权
          </p>
        </template>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import {ref} from 'vue'
import {Icon} from '@iconify/vue'
import {NButton, NSwitch, NInput} from "naive-ui";
import {useAuthorizationStore} from '../stores/Authorization.ts'
import {url} from "../services/UrlService";

defineProps({
  show: Boolean,
});

const store = useAuthorizationStore();
const isLogin = ref(store.isAuthenticated);

// 使用store中的getter获取状态
const isShow = ref(store.getIsShowEdu);
const userData = ref(store.getUserData);
const eduPassword = ref(store.getEduPassword);

const handleChange = (value) => {
  // 使用store中的action来更新状态
  store.setIsShowEdu(value);
  isShow.value = value;
}

const handleGetData = async () => {
  // 先更新store中的密码
  store.setEduPassword(eduPassword.value);
  // 使用store中的action来获取教育数据
  await store.handleGetEduData();
}

const emit = defineEmits(['update:show']);

function closeModal() {
  emit('update:show', false);
}

// 处理OAuth授权登录
const handleOAuthLogin = () => {
  window.location.href = `${url}/auth/authorize`;
}
</script>

<style scoped>
.sheet-backdrop {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  background: rgba(0, 0, 0, 0.22);
  -webkit-backdrop-filter: blur(18px);
  backdrop-filter: blur(18px);
}

.dark .sheet-backdrop {
  background: rgba(0, 0, 0, 0.5);
}

.sheet-panel {
  width: 100%;
  max-width: 440px;
  padding: 26px;
  border-radius: 28px;
  border: 1px solid var(--glass-border);
  background: var(--glass-strong);
  box-shadow: 0 32px 80px rgba(15, 23, 42, 0.24);
  -webkit-backdrop-filter: blur(32px) saturate(180%);
  backdrop-filter: blur(32px) saturate(180%);
  animation: sheet-in 0.35s var(--ease-apple);
}

.dark .sheet-panel {
  box-shadow: 0 32px 80px rgba(0, 0, 0, 0.75);
}

@keyframes sheet-in {
  from {
    opacity: 0;
    transform: translateY(10px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.close-btn {
  display: flex;
  width: 32px;
  height: 32px;
  flex: none;
  align-items: center;
  justify-content: center;
  border: 0;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.055);
  color: var(--ink-2);
  cursor: pointer;
  transition: background-color 0.2s ease, color 0.2s ease;
}

.dark .close-btn {
  background: rgba(255, 255, 255, 0.08);
}

.close-btn:hover {
  color: var(--ink);
}

.close-btn svg {
  width: 15px;
  height: 15px;
}

.setting-row {
  display: flex;
  min-height: 64px;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 12px 0;
}

.setting-label {
  font-size: 15px;
  font-weight: 500;
  color: var(--ink);
}

.setting-hint {
  margin-top: 2px;
  font-size: 12.5px;
  color: var(--ink-2);
}

.password-input {
  width: 56%;
}

.primary-btn {
  width: 100%;
  height: 48px;
  margin-top: 20px;
  border: 0;
  border-radius: 999px;
  background: var(--accent);
  color: #fff;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: filter 0.2s ease, transform 0.15s ease;
}

.primary-btn:hover {
  filter: brightness(1.07);
}

.primary-btn:active {
  transform: scale(0.98);
}

@media (prefers-reduced-motion: reduce) {
  .sheet-panel {
    animation: none;
  }
}
</style>
