<template>
  <span class="app-logo" :style="{ width: `${size}px`, height: `${size}px` }">
    <span v-if="failed" class="app-logo-fallback" :style="fallbackStyle">
      {{ fallbackText }}
    </span>
    <template v-else>
      <img
          v-if="imageSrc"
          :src="imageSrc"
          :alt="name"
          class="app-logo-img"
          loading="lazy"
          @error="failed = true"
          @load="failed = false"
      />
      <IconFont v-else :type="`#icon-${icon}`" :size="size" />
    </template>
  </span>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';
import IconFont from './IconFont.vue';
import { getIcon } from '../utils/urlHelper';

const props = withDefaults(defineProps<{
    url?: string;
    name?: string;
    icon?: string;
    size?: number;
}>(), {
    url: '',
    name: '',
    icon: '',
    size: 40,
});

const failed = ref(false)
let attempts = 0
let timer: number | undefined

const imageSrc = computed(() => {
  if (props.icon) {
    return props.icon.startsWith('http') ? props.icon : ''
  }
  return getIcon(props.url)
})

const fallbackText = computed(() => {
  const text = (props.name || '').trim()
  return text ? text.charAt(0).toUpperCase() : '·'
})

const palette = ['#0071e3', '#bf5af2', '#ff9f0a', '#30d158', '#ff375f', '#40c8e0', '#5e5ce6']

const fallbackStyle = computed(() => {
  let hash = 0
  for (const char of props.name || props.url || '') {
    hash = (hash * 31 + char.charCodeAt(0)) | 0
  }
  const color = palette[Math.abs(hash) % palette.length]
  return {
    background: `color-mix(in srgb, ${color} 16%, transparent)`,
    color,
    fontSize: `${Math.max(10, Math.round(props.size * 0.42))}px`,
  }
})

// iconfont 图标缺失时用首字母兜底
const checkIconfont = () => {
  if (document.getElementById(`icon-${props.icon}`)) {
    failed.value = false
    attempts = 99
    return
  }
  attempts += 1
  if (attempts >= 4) {
    failed.value = true
  }
}

const watchIconfont = () => {
  if (timer) clearInterval(timer)
  if (!props.icon || props.icon.startsWith('http')) return

  checkIconfont()
  timer = window.setInterval(() => {
    if (attempts >= 4) {
      clearInterval(timer)
      return
    }
    checkIconfont()
  }, 400)
}

watch(() => [props.icon, props.url], () => {
  failed.value = false
  attempts = 0
  watchIconfont()
})

onMounted(() => {
  watchIconfont()
})

onUnmounted(() => {
  if (timer) clearInterval(timer)
})
</script>

<style scoped>
.app-logo {
  display: inline-flex;
  flex: none;
  align-items: center;
  justify-content: center;
}

.app-logo-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 6px;
}

.app-logo-fallback {
  display: flex;
  width: 100%;
  height: 100%;
  align-items: center;
  justify-content: center;
  border-radius: 32%;
  font-weight: 600;
  line-height: 1;
  letter-spacing: 0;
}
</style>
