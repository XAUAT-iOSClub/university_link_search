<script setup lang="ts">
import { computed } from 'vue';
import { checkIsWeiXin } from '../../utils/urlHelper';
import AppLogo from '../AppLogo.vue';

interface Link {
  url: string;
  description: string;
  name: string;
  icon: string;
}

const props = defineProps<{
  link: Link;
}>();

const emit = defineEmits<{
  (e: 'open-qr', link: Link): void
}>();

const isWeiXin = checkIsWeiXin();

const isQrMode = computed(() => {
  return !isWeiXin && props.link.description === '微信打开';
});

const handleClick = (e: MouseEvent) => {
  if (isQrMode.value) {
    e.preventDefault();
    emit('open-qr', props.link);
  }
};
</script>

<template>
  <a
      :href="isQrMode ? undefined : link.url"
      :target="isQrMode ? undefined : '_blank'"
      :rel="isQrMode ? undefined : 'noopener noreferrer'"
      :title="link.description || link.name"
      class="link-item group"
      @click="handleClick"
  >
    <span class="link-icon">
      <AppLogo :url="link.url" :name="link.name" :icon="link.icon" :size="30" />
    </span>
    <span class="link-name">{{ link.name }}</span>
  </a>
</template>

<style scoped>
.link-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 4px 2px;
  border-radius: 16px;
  text-decoration: none;
  text-align: center;
  outline: none;
}

.link-icon {
  display: flex;
  width: 48px;
  height: 48px;
  align-items: center;
  justify-content: center;
  border-radius: 15px;
  background: rgba(255, 255, 255, 0.72);
  border: 1px solid rgba(255, 255, 255, 0.85);
  box-shadow: 0 1px 2px rgba(15, 23, 42, 0.05), 0 6px 16px rgba(15, 23, 42, 0.05);
  transition:
    transform 0.35s var(--ease-apple),
    box-shadow 0.35s var(--ease-apple),
    background-color 0.3s ease;
}

.dark .link-icon {
  background: rgba(255, 255, 255, 0.07);
  border-color: rgba(255, 255, 255, 0.09);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.06);
}

.link-name {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  width: 100%;
  font-size: 12.5px;
  line-height: 1.3;
  color: var(--ink-2);
  transition: color 0.2s ease;
}

.dark .link-name {
  color: rgba(255, 255, 255, 0.58);
}

.link-item:hover .link-icon {
  transform: translateY(-3px) scale(1.04);
  box-shadow: 0 10px 24px rgba(15, 23, 42, 0.14);
}

.dark .link-item:hover .link-icon {
  background: rgba(255, 255, 255, 0.12);
  box-shadow: 0 10px 24px rgba(0, 0, 0, 0.45);
}

.link-item:hover .link-name {
  color: var(--accent);
}

.link-item:active .link-icon {
  transform: scale(0.94);
  transition-duration: 0.12s;
}

.link-item:focus-visible .link-icon {
  box-shadow: 0 0 0 3px var(--accent-soft), 0 6px 16px rgba(15, 23, 42, 0.08);
}

@media (hover: none) {
  .link-item:hover .link-icon {
    transform: none;
  }
}
</style>
