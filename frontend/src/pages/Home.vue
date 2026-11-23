<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import QrCodeModal from "../components/QrCodeModal.vue";
import TilesCard from "../components/TilesCard.vue";
import IconFont from "../components/IconFont.vue";
import AppleCard from "../components/AppleCard.vue";
import FooterContent from "../components/FooterContent.vue";
import SearchBar from "../components/SearchBar.vue";
import LinkCard from "../components/home/LinkCard.vue";
import { CategoryService } from "../services/CategoryService";
import { useSettingStore } from "../stores/SettingStore";

const categories = ref<any[]>([])
const showQrCodeModal = ref(false);
const currentModalLink = ref(null);
const searchRef = ref()
const settingStore = useSettingStore()

const totalLinks = computed(() =>
  categories.value.reduce((total, category) => total + (category.links?.length || 0), 0)
)

const qrCodeModalOpen = (link: any) => {
  currentModalLink.value = link;
  showQrCodeModal.value = true;
}

// 内容量决定卡片跨度：链接多的分类占 2/3 宽，其余占 1/3
const categorySpanClass = (category: any): string => {
  return (category.links?.length || 0) >= 10
    ? 'col-span-12 md:col-span-6 lg:col-span-8'
    : 'col-span-12 md:col-span-6 lg:col-span-4'
}

const linkGridClass = (category: any): string => {
  return (category.links?.length || 0) >= 10
    ? 'grid grid-cols-4 gap-x-1 gap-y-4 lg:grid-cols-5'
    : 'grid grid-cols-4 gap-x-1 gap-y-4'
}

const tints = ['#0a84ff', '#bf5af2', '#ff9f0a', '#30d158', '#ff375f', '#40c8e0', '#5e5ce6']

const tintStyle = (index: number) => {
  const color = tints[index % tints.length]
  return {
    color,
    backgroundColor: `color-mix(in srgb, ${color} 15%, transparent)`,
  }
}

onMounted(async () => {
  categories.value = await CategoryService.getAllCategories()
})
</script>

<template>
  <div class="relative min-h-screen overflow-x-hidden">
    <!-- Sonoma 氛围背景 -->
    <div class="sonoma-canvas" aria-hidden="true">
      <span class="aurora aurora-1"></span>
      <span class="aurora aurora-2"></span>
      <span class="aurora aurora-3"></span>
      <span class="grain"></span>
    </div>

    <!-- 顶栏 + Hero 搜索 -->
    <SearchBar ref="searchRef" />

    <main class="relative z-10 mx-auto w-full max-w-[1240px] px-5 pb-4 pt-2 md:px-8">
      <!-- 待办 / 剪贴板磁贴 -->
      <div v-if="settingStore.showTiles" class="mb-5 md:mb-6">
        <TilesCard class="apple-reveal" />
      </div>

      <template v-if="categories.length > 0">
        <!-- 区块标题 -->
        <div class="apple-reveal mb-6 flex items-end justify-between gap-4">
          <div>
            <h2 class="text-[26px] font-semibold tracking-[-0.02em] text-[var(--ink)] md:text-[30px]">
              全部服务
            </h2>
          </div>
        </div>

        <!-- 分类 Bento 栅格 -->
        <div class="grid grid-cols-12 gap-4 lg:gap-6">
          <div
            v-for="(category, index) in categories"
            :key="category.key || index"
            :class="[categorySpanClass(category), 'apple-reveal']"
            :style="{ '--delay': `${Math.min(index * 60, 320)}ms` }"
          >
            <AppleCard class="h-full">
              <template #title>
                <div class="flex items-center gap-3.5">
                  <span class="cat-icon" :style="tintStyle(index)">
                    <IconFont :type="`#icon-${category.icon}`" :size="24" />
                  </span>
                  <div class="min-w-0">
                    <h3 class="truncate text-[19px] font-semibold tracking-[-0.01em] text-[var(--ink)]">
                      {{ category.name }}
                    </h3>
                    <p class="mt-0.5 truncate text-[13px] text-[var(--ink-2)]">
                      {{ category.description }}
                    </p>
                  </div>
                </div>
              </template>

              <template #context>
                <div :class="linkGridClass(category)" class="mt-5">
                  <LinkCard
                    v-for="link in category.links"
                    :key="link.key || link.url"
                    :link="link"
                    @open-qr="qrCodeModalOpen"
                  />
                </div>
              </template>
            </AppleCard>
          </div>
        </div>
      </template>
    </main>

    <FooterContent />

    <QrCodeModal v-model:show="showQrCodeModal" :currentLink="currentModalLink" />
  </div>
</template>

<style scoped>
.sonoma-canvas {
  position: fixed;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
  background:
    radial-gradient(120% 90% at 12% -5%, #eaf2ff 0%, transparent 58%),
    radial-gradient(110% 85% at 92% 4%, #ffeef4 0%, transparent 52%),
    linear-gradient(165deg, #fbfbfd 0%, #f2f4f9 52%, #fdf4f7 100%);
}

.dark .sonoma-canvas {
  background:
    radial-gradient(120% 90% at 12% -5%, rgba(10, 132, 255, 0.20) 0%, transparent 58%),
    radial-gradient(110% 85% at 92% 4%, rgba(191, 90, 242, 0.16) 0%, transparent 52%),
    linear-gradient(165deg, #030407 0%, #070b14 55%, #0d0714 100%);
}

.aurora {
  position: absolute;
  border-radius: 50%;
  filter: blur(80px);
  opacity: 0.42;
  will-change: transform;
  animation: aurora-drift 32s ease-in-out infinite alternate;
}

.aurora-1 {
  width: 46vw;
  height: 46vw;
  min-width: 420px;
  min-height: 420px;
  top: -16vw;
  left: -8vw;
  background: radial-gradient(circle, rgba(122, 168, 255, 0.9) 0%, rgba(122, 168, 255, 0) 68%);
}

.aurora-2 {
  width: 40vw;
  height: 40vw;
  min-width: 380px;
  min-height: 380px;
  top: -12vw;
  right: -8vw;
  background: radial-gradient(circle, rgba(255, 168, 200, 0.85) 0%, rgba(255, 168, 200, 0) 68%);
  animation-delay: -10s;
}

.aurora-3 {
  width: 48vw;
  height: 48vw;
  min-width: 420px;
  min-height: 420px;
  bottom: -28vw;
  left: 18vw;
  background: radial-gradient(circle, rgba(196, 181, 253, 0.8) 0%, rgba(196, 181, 253, 0) 68%);
  opacity: 0.3;
  animation-delay: -18s;
}

.dark .aurora {
  opacity: 0.5;
}

.dark .aurora-1 {
  background: radial-gradient(circle, rgba(10, 132, 255, 0.75) 0%, rgba(10, 132, 255, 0) 68%);
}

.dark .aurora-2 {
  background: radial-gradient(circle, rgba(191, 90, 242, 0.65) 0%, rgba(191, 90, 242, 0) 68%);
}

.dark .aurora-3 {
  background: radial-gradient(circle, rgba(94, 92, 230, 0.6) 0%, rgba(94, 92, 230, 0) 68%);
  opacity: 0.38;
}

.grain {
  position: absolute;
  inset: 0;
  opacity: 0.03;
  mix-blend-mode: overlay;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.5'/%3E%3C/svg%3E");
  background-size: 160px 160px;
}

@keyframes aurora-drift {
  0% {
    transform: translate3d(0, 0, 0) scale(1);
  }
  50% {
    transform: translate3d(2.5%, 3%, 0) scale(1.06);
  }
  100% {
    transform: translate3d(-2%, 1.5%, 0) scale(0.98);
  }
}

.cat-icon {
  display: flex;
  width: 44px;
  height: 44px;
  flex: none;
  align-items: center;
  justify-content: center;
  border-radius: 14px;
}

@media (max-width: 640px) {
  .aurora {
    animation: none;
    filter: blur(60px);
    opacity: 0.3;
  }

  .grain {
    display: none;
  }
}

@media (prefers-reduced-motion: reduce) {
  .aurora {
    animation: none;
  }
}
</style>
