<template>
  <div
      :id="id"
      :style="style"
      :class="['apple-card', customClass]"
  >
    <div
        v-if="$slots.title"
        :class="['apple-card-header', headerClass]"
        :style="headerStyle"
    >
      <slot name="title"></slot>
    </div>

    <div
        :class="['apple-card-context', contextClass]"
        :style="contextStyle"
    >
      <slot name="context"></slot>
    </div>
  </div>
</template>

<script setup>
defineProps({
  id: String,
  style: String,
  customClass: {
    type: String,
    default: ''
  },
  contextStyle: String,
  headerStyle: String,
  headerClass: String,
  contextClass: String
})
</script>

<style scoped>
.apple-card {
  position: relative;
  border-radius: 28px;
  background: var(--glass);
  border: 1px solid var(--glass-border);
  box-shadow: var(--shadow-card);
  -webkit-backdrop-filter: blur(24px) saturate(180%);
  backdrop-filter: blur(24px) saturate(180%);
  transition:
    transform 0.4s var(--ease-apple),
    box-shadow 0.4s var(--ease-apple),
    background-color 0.4s ease;
  overflow: hidden;
}

@media screen and (min-width: 768px) {
  .apple-card:hover {
    transform: translateY(-3px);
    box-shadow: var(--shadow-card-hover);
  }
}

@media screen and (hover: none) {
  .apple-card:hover {
    transform: none;
  }
}

.apple-card-header {
  padding: 22px 22px 0;
}

.apple-card-context {
  padding: 0 22px 22px;
}

@media screen and (max-width: 640px) {
  .apple-card {
    border-radius: 24px;
  }

  .apple-card-header {
    padding: 18px 18px 0;
  }

  .apple-card-context {
    padding: 0 18px 18px;
  }
}

@media (prefers-reduced-transparency: reduce) {
  .apple-card {
    background: color-mix(in srgb, var(--canvas) 88%, white 12%);
    -webkit-backdrop-filter: none;
    backdrop-filter: none;
  }

  .dark .apple-card {
    background: #1c1c1e;
  }
}
</style>
