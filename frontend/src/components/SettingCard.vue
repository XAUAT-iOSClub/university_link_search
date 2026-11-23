<template>
  <Teleport to="body">
    <div v-if="show" class="sheet-backdrop" @click="closeModal">
      <div class="sheet-panel" @click.stop>
        <div class="flex items-start justify-between gap-4">
          <div>
            <h2 class="text-[22px] font-semibold tracking-[-0.02em] text-[var(--ink)]">应用设置</h2>
            <p class="mt-1 text-[13px] text-[var(--ink-2)]">外观与显示选项</p>
          </div>
          <button class="close-btn" aria-label="关闭" @click="closeModal">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round">
              <path d="M6 6l12 12M18 6L6 18" />
            </svg>
          </button>
        </div>

        <div class="mt-5 divide-y divide-[var(--hairline)]">
          <div class="setting-row">
            <div>
              <p class="setting-label">显示磁贴</p>
              <p class="setting-hint">在首页展示待办与剪贴板</p>
            </div>
            <n-switch v-model:value="isShow" @update:value="handleChange"/>
          </div>

          <div class="setting-row">
            <div>
              <p class="setting-label">深色模式</p>
              <p class="setting-hint">跟随你的使用习惯切换外观</p>
            </div>
            <n-switch v-model:value="isDarkMode" />
          </div>
        </div>

        <button
          v-if="authStore.isAuthenticated"
          class="logout-btn"
          @click="handleLogout"
        >
          退出登录
        </button>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import {NSwitch, useMessage} from "naive-ui";
import {computed} from "vue";
import {useSettingStore} from "../stores/SettingStore";
import {useAuthorizationStore} from "../stores/Authorization";
import {useThemeStore} from "../stores/ThemeStore";
import {storeToRefs} from "pinia";

defineProps({
  show: Boolean,
});

const emit = defineEmits(['update:show']);
const settingStore = useSettingStore();
const authStore = useAuthorizationStore();
const themeStore = useThemeStore();
const { theme } = storeToRefs(themeStore);
const message = useMessage();

function closeModal() {
  emit('update:show', false);
}

const isShow = computed({
  get: () => settingStore.showTiles,
  set: (value) => settingStore.setShowTiles(value)
})

const isDarkMode = computed({
  get: () => theme.value === 'dark',
  set: (val) => theme.value = val ? 'dark' : 'light'
})

const handleChange = (value) => {
  settingStore.setShowTiles(value);
}

const handleLogout = () => {
  authStore.logout();
  message.success("已退出登录");
  closeModal();
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

.logout-btn {
  width: 100%;
  height: 46px;
  margin-top: 18px;
  border: 0;
  border-radius: 999px;
  background: rgba(255, 59, 48, 0.12);
  color: #ff3b30;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s ease, transform 0.15s ease;
}

.logout-btn:hover {
  background: rgba(255, 59, 48, 0.18);
}

.logout-btn:active {
  transform: scale(0.98);
}

@media (prefers-reduced-motion: reduce) {
  .sheet-panel {
    animation: none;
  }
}
</style>
