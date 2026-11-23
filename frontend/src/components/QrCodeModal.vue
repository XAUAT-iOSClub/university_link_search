<template>
  <Teleport to="body">
    <div v-if="show" class="sheet-backdrop" @click="closeModal">
      <div class="sheet-panel qr-panel" @click.stop>
        <div class="flex items-center gap-3">
          <span v-if="currentLink?.icon" class="qr-icon">
            <IconFont :type="`#icon-${currentLink.icon}`" :size="26" />
          </span>
          <div class="min-w-0 flex-1">
            <h2 class="truncate text-[20px] font-semibold tracking-[-0.01em] text-[var(--ink)]">
              {{ currentLink?.name || '扫码打开' }}
            </h2>
            <p class="mt-0.5 text-[12.5px] text-[var(--ink-2)]">请使用微信扫码打开</p>
          </div>
          <button class="close-btn" aria-label="关闭" @click="closeModal">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round">
              <path d="M6 6l12 12M18 6L6 18" />
            </svg>
          </button>
        </div>

        <div class="mt-6 flex justify-center">
          <div class="qr-box">
            <img :src="qrCodeUrl" alt="二维码" class="h-56 w-56" />
          </div>
        </div>

        <button class="primary-btn" @click="closeModal">完成</button>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import {ref, watch} from 'vue';
import QRCode from 'qrcode';
import IconFont from "./IconFont.vue";

const props = defineProps({
  show: Boolean,
  currentLink: Object
});

const emit = defineEmits(['update:show']);

const qrCodeUrl = ref('');

// 当链接变化时生成新的二维码
watch(() => props.currentLink, async (newLink) => {
  if (newLink?.url) {
    try {
      qrCodeUrl.value = await QRCode.toDataURL(newLink.url, {
        margin: 1,
        width: 448,
        color: { dark: '#1d1d1f', light: '#ffffff' }
      });
    } catch (err) {
      console.error('QR Code generation error:', err);
    }
  }
}, {immediate: true});

function closeModal() {
  emit('update:show', false);
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
  max-width: 380px;
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

.qr-icon {
  display: flex;
  width: 44px;
  height: 44px;
  flex: none;
  align-items: center;
  justify-content: center;
  border-radius: 14px;
  background: var(--accent-soft);
  color: var(--accent);
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

.qr-box {
  padding: 14px;
  border-radius: 20px;
  background: #fff;
  box-shadow: 0 8px 24px rgba(15, 23, 42, 0.08);
}

.primary-btn {
  width: 100%;
  height: 48px;
  margin-top: 24px;
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
