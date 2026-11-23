<template>
  <!-- ===== 顶栏：始终可见 ===== -->
  <header class="site-header" :class="{ 'site-header-scrolled': isScrolled }">
    <div class="mx-auto flex h-14 w-full max-w-[1240px] items-center justify-between px-5 md:h-16 md:px-8">
      <!-- 品牌 -->
      <router-link to="/" class="group flex items-center gap-2.5" aria-label="建大导航首页">
        <span class="brand-mark">
          <img src="/logo.png" alt="" />
        </span>
        <span class="hidden text-[15px] font-semibold tracking-tight text-[var(--ink)] sm:block">
          建大导航
        </span>
      </router-link>

      <!-- 吸顶后的紧凑搜索 -->
      <div class="hidden flex-1 justify-center px-8 md:flex">
        <Transition name="compact">
          <div v-if="showCompactSearch" class="compact-search">
            <Icon icon="solar:magnifer-linear" class="h-4 w-4 shrink-0 text-[var(--ink-2)]" />
            <input
              v-model="searchQuery"
              type="text"
              :placeholder="`在${currentEngine.label}中搜索`"
              class="min-w-0 flex-1 bg-transparent text-[13.5px] text-[var(--ink)] outline-none placeholder:text-[var(--ink-2)]"
              aria-label="搜索"
              @focus="closePanels"
              @keyup.enter="search"
            />
            <button class="compact-go" aria-label="搜索" @click="search">
              <Icon icon="solar:arrow-right-linear" class="h-4 w-4" />
            </button>
          </div>
        </Transition>
      </div>

      <!-- 右侧操作 -->
      <div class="flex items-center gap-0.5 sm:gap-1">
        <button
          class="header-action"
          :aria-label="isDarkMode ? '切换到浅色模式' : '切换到深色模式'"
          @click="toggleTheme"
        >
          <Icon :icon="isDarkMode ? 'solar:sun-bold' : 'solar:moon-bold'" class="h-[18px] w-[18px]" />
        </button>

        <button class="header-action md:hidden" aria-label="搜索" @click="focusHeroSearch">
          <Icon icon="solar:magnifer-bold" class="h-[18px] w-[18px]" />
        </button>

        <div
          class="relative"
          @mouseenter="isOpenAccount = true"
          @mouseleave="isOpenAccount = false"
        >
          <button
            class="header-action"
            aria-label="账户"
            @click="isOpenAccount = false; closePanels(); isOpenLogin = true"
          >
            <Icon icon="solar:user-circle-bold" class="h-[18px] w-[18px]" />
          </button>
          <Transition name="drop">
            <div v-if="isOpenAccount" class="account-pop">
              <AccountCard />
            </div>
          </Transition>
        </div>

        <button
          id="setting"
          class="header-action"
          aria-label="设置"
          @click="closePanels(); isOpenSetting = true"
        >
          <Icon icon="solar:settings-bold" class="h-[18px] w-[18px]" />
        </button>
      </div>
    </div>
  </header>

  <!-- ===== Hero：时钟 + 搜索 ===== -->
  <section class="relative z-30 px-5 pb-14 pt-32 md:px-8 md:pb-20 md:pt-40">
    <div class="mx-auto flex max-w-[760px] flex-col items-center text-center">
      <p class="hero-clock">{{ time }}</p>
      <p class="mt-3 text-[15px] font-medium text-[var(--ink-2)]">
        {{ greeting }}，欢迎回来
      </p>
      <p class="mt-1 text-[13px] text-[var(--ink-2)] opacity-70">{{ dateLabel }}</p>

      <div ref="heroSearch" class="relative mt-9 w-full max-w-[660px]">
        <div
          ref="searchWrap"
          class="search-shell"
          :class="{ 'search-shell-focused': isSearchFocused }"
        >
          <!-- 搜索引擎选择器 -->
          <div ref="engineWrap" class="relative" id="chooseEngine">
            <button
              class="engine-btn"
              :aria-label="`当前搜索引擎：${currentEngine.label}`"
              @click.stop="showEngines = !showEngines"
            >
              <component :is="currentEngine.icon()" class="h-5 w-5 rounded" />
              <Icon icon="solar:alt-arrow-down-linear" class="h-3.5 w-3.5 opacity-60" />
            </button>

            <Transition name="drop">
              <div v-if="showEngines" class="engine-panel">
                <button
                  v-for="engine in searchEngines"
                  :key="engine.key"
                  class="panel-row"
                  :class="{ 'panel-row-active': engine.key === currentEngine.key }"
                  @click="selectEngine(engine)"
                >
                  <component :is="engine.icon()" class="h-5 w-5 rounded" />
                  <span class="flex-1 text-left">{{ engine.label }}</span>
                  <Icon
                    v-if="engine.key === currentEngine.key"
                    icon="solar:check-circle-bold"
                    class="h-4 w-4 text-[var(--accent)]"
                  />
                </button>
              </div>
            </Transition>
          </div>

          <span class="h-6 w-px shrink-0 bg-[var(--hairline)]"></span>

          <!-- 搜索输入 -->
          <input
            ref="searchInput"
            v-model="searchQuery"
            type="text"
            :placeholder="`在${currentEngine.label}中搜索`"
            aria-label="搜索"
            class="search-input"
            @keyup="getSuggestions"
            @keyup.enter="search"
            @keydown.down.prevent="moveSuggestion(1)"
            @keydown.up.prevent="moveSuggestion(-1)"
            @keydown.esc="closePanels"
            @focus="isSearchFocused = true"
            @blur="isSearchFocused = false"
          />

          <!-- 搜索按钮 -->
          <button class="search-submit" aria-label="搜索" @click="search">
            <Icon icon="solar:magnifer-bold" class="h-[18px] w-[18px]" />
          </button>

          <!-- 搜索建议 -->
          <Transition name="drop">
            <div v-if="isOpenSuggestions" class="suggest-panel" @mousedown.prevent>
              <button class="panel-row" @click="transform">
                <Icon icon="solar:translation-bold" class="h-[18px] w-[18px] text-[var(--accent)]" />
                <span class="flex-1 truncate text-left">翻译：{{ searchQuery }}</span>
                <span class="text-[11px] text-[var(--ink-2)]">翻译</span>
              </button>

              <div class="my-1 h-px bg-[var(--hairline)]"></div>

              <button
                v-for="(suggestion, index) in suggestions"
                :key="suggestion"
                class="panel-row"
                :class="{ 'panel-row-active': index === selectedSuggestionIndex }"
                @click="openUrl(suggestion)"
              >
                <Icon icon="solar:magnifer-linear" class="h-[18px] w-[18px] text-[var(--ink-2)]" />
                <span class="flex-1 truncate text-left">{{ suggestion }}</span>
              </button>
            </div>
          </Transition>
        </div>
      </div>
    </div>
  </section>

  <SettingCard v-model:show="isOpenSetting" />
  <LoginCard v-model:show="isOpenLogin" />
</template>

<script setup lang="ts">
import { computed, h, onMounted, onUnmounted, ref, watch } from 'vue'
import { Icon } from '@iconify/vue'
import AccountCard from "./AccountCard.vue";
import SettingCard from "./SettingCard.vue";
import LoginCard from "./LoginCard.vue";
import { useSettingStore } from '../stores/SettingStore'
import { useThemeStore } from '../stores/ThemeStore'
import { storeToRefs } from 'pinia'

// 定义搜索引擎类型
interface SearchEngine {
  key: string;
  label: string;
  icon: any;
}

// 使用SettingStore
const settingStore = useSettingStore();
const themeStore = useThemeStore();
const { isDarkMode } = storeToRefs(themeStore);
const toggleTheme = themeStore.toggleTheme;

// 搜索引擎图标使用 Iconify 品牌图标，避免远程 favicon 失效
function renderIcon(icon: string, color: string): () => any {
  return () => {
    return h(Icon, { icon, class: 'h-5 w-5', style: { color } });
  };
}

const engineLink = ['https://www.baidu.com/s?wd=', 'https://www.google.com/search?q=', 'https://www.bing.com/search?q=', 'https://duckduckgo.com/?q=']

const searchEngines: SearchEngine[] = [
  {
    key: '1',
    label: '百度',
    icon: renderIcon('simple-icons:baidu', '#2932e1'),
  },
  {
    key: '2',
    label: '谷歌',
    icon: renderIcon('simple-icons:google', '#4285f4'),
  },
  {
    key: '3',
    label: '必应',
    icon: renderIcon('simple-icons:microsoftbing', '#008373'),
  },
  {
    key: '4',
    label: 'DuckDuckGo',
    icon: renderIcon('simple-icons:duckduckgo', '#de5833'),
  }
]

const now = ref(new Date())
const searchQuery = ref('')
const showEngines = ref(false)
const suggestions = ref<string[]>([])
const selectedSuggestionIndex = ref(-1)
const isOpenSuggestions = ref(false)
const isOpenSetting = ref(false)
const isOpenAccount = ref(false)
const isOpenLogin = ref(false)
const isSearchFocused = ref(false)
const isScrolled = ref(false)
const showCompactSearch = ref(false)

const currentEngine = ref<SearchEngine>(searchEngines[0])

const heroSearch = ref<HTMLElement | null>(null)
const searchInput = ref<HTMLInputElement | null>(null)
const searchWrap = ref<HTMLElement | null>(null)
const engineWrap = ref<HTMLElement | null>(null)

const pad = (value: number) => value.toString().padStart(2, '0')

const time = computed(() => `${pad(now.value.getHours())}:${pad(now.value.getMinutes())}`)

const greeting = computed(() => {
  const hour = now.value.getHours()
  if (hour < 5) return '夜深了'
  if (hour < 9) return '早上好'
  if (hour < 12) return '上午好'
  if (hour < 14) return '中午好'
  if (hour < 18) return '下午好'
  return '晚上好'
})

const dateLabel = computed(() =>
  new Intl.DateTimeFormat('zh-CN', { month: 'long', day: 'numeric', weekday: 'long' }).format(now.value)
)

// Watch for changes in the store
watch(() => settingStore.getCurrentEngineKey, (newKey: string) => {
  const engine = searchEngines.find(item => item.key === newKey);
  if (engine) {
    currentEngine.value = engine;
  }
});

// Initialize current engine from store
const initializeCurrentEngine = (): void => {
  settingStore.initCurrentEngine();
  const engine = searchEngines.find(item => item.key === settingStore.getCurrentEngineKey);
  if (engine) {
    currentEngine.value = engine;
  }
};

const selectEngine = (engine: string | SearchEngine): void => {
  const engineKey = typeof engine === 'string' ? engine : engine.key;
  const foundEngine = searchEngines.find(item => item.key === engineKey);
  if (foundEngine) {
    currentEngine.value = foundEngine;
  }
  showEngines.value = false;

  settingStore.setCurrentEngine(engineKey);
};

const search = async (): Promise<void> => {
  if (selectedSuggestionIndex.value !== -1 && selectedSuggestionIndex.value < suggestions.value.length) {
    const engineIndex = parseInt(currentEngine.value.key) - 1;
    if (engineIndex >= 0 && engineIndex < engineLink.length) {
      const searchUrl = engineLink[engineIndex] + encodeURIComponent(suggestions.value[selectedSuggestionIndex.value]);
      window.open(searchUrl);
    }
    searchQuery.value = '';
    selectedSuggestionIndex.value = -1;
    isOpenSuggestions.value = false;
    return;
  }

  if (searchQuery.value.trim()) {
    const engineIndex = parseInt(currentEngine.value.key) - 1;
    if (engineIndex >= 0 && engineIndex < engineLink.length) {
      const searchUrl = engineLink[engineIndex] + encodeURIComponent(searchQuery.value);
      window.open(searchUrl);
    }
  }
};

const transform = (): void => {
  if (searchQuery.value.trim()) {
    let searchUrl = '';
    switch (currentEngine.value.key) {
      case '1':
        searchUrl = `https://fanyi.baidu.com/#auto/en/${encodeURIComponent(searchQuery.value)}`;
        break;
      case '2':
      case '3':
      case '4':
        searchUrl = `https://translate.google.com/?sl=auto&tl=en&text=${encodeURIComponent(searchQuery.value)}&op=translate`;
        break;
    }

    window.open(searchUrl);
    isOpenSuggestions.value = false;
  }
};

// 定义百度建议接口的响应类型
interface BaiduSuggestionResponse {
  s: string[];
}

const getSuggestions = async (): Promise<void> => {
  selectedSuggestionIndex.value = -1;

  if (searchQuery.value === '') {
    isOpenSuggestions.value = false;
    suggestions.value = [];
    return;
  }

  const result = await new Promise<BaiduSuggestionResponse>((resolve) => {
    const callbackName = `jsonp_${Date.now()}_${Math.floor(Math.random() * 100000)}`;
    (window as any)[callbackName] = (data: BaiduSuggestionResponse) => {
      resolve(data);
      delete (window as any)[callbackName];
      document.body.removeChild(script);
    };
    const script = document.createElement('script');
    script.src = `https://suggestion.baidu.com/su?wd=${encodeURIComponent(searchQuery.value)}&cb=${callbackName}`;
    document.body.appendChild(script);
  });

  suggestions.value = result.s || [];
  isOpenSuggestions.value = suggestions.value.length > 0;
};

const moveSuggestion = (step: number): void => {
  if (!isOpenSuggestions.value || suggestions.value.length === 0) return
  const next = selectedSuggestionIndex.value + step
  selectedSuggestionIndex.value = Math.max(-1, Math.min(next, suggestions.value.length - 1))
}

const closePanels = (): void => {
  showEngines.value = false
  isOpenSuggestions.value = false
}

const openUrl = (text: string): void => {
  const engineIndex = parseInt(currentEngine.value.key) - 1;
  if (engineIndex >= 0 && engineIndex < engineLink.length) {
    const searchUrl = engineLink[engineIndex] + encodeURIComponent(text);
    window.open(searchUrl);
  }
  isOpenSuggestions.value = false
};

const focusHeroSearch = (): void => {
  heroSearch.value?.scrollIntoView({ behavior: 'smooth', block: 'center' })
  window.setTimeout(() => searchInput.value?.focus(), 260)
}

const handleScroll = (): void => {
  isScrolled.value = window.scrollY > 8
  const hero = heroSearch.value
  showCompactSearch.value = !!hero && hero.getBoundingClientRect().bottom < 120
}

// 点击外部区域关闭菜单
const handleClickOutside = (event: MouseEvent): void => {
  const target = event.target as Node
  if (engineWrap.value && !engineWrap.value.contains(target)) {
    showEngines.value = false
  }
  if (searchWrap.value && !searchWrap.value.contains(target)) {
    isOpenSuggestions.value = false
  }
}

// 右键仍保留为打开设置的快捷方式
const handleContextMenu = (event: MouseEvent): void => {
  const target = event.target as HTMLElement
  if (
    target.tagName === 'INPUT' ||
    target.tagName === 'TEXTAREA' ||
    target.isContentEditable ||
    target.closest('#chooseEngine')
  ) {
    return
  }
  event.preventDefault()
  closePanels()
  isOpenSetting.value = true
}

let clockTimer: number | undefined

onMounted(() => {
  initializeCurrentEngine();
  handleScroll()
  window.addEventListener('scroll', handleScroll, { passive: true });
  window.addEventListener('click', handleClickOutside);
  document.addEventListener('contextmenu', handleContextMenu);
  clockTimer = window.setInterval(() => {
    now.value = new Date()
  }, 30000);
});

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll);
  window.removeEventListener('click', handleClickOutside);
  document.removeEventListener('contextmenu', handleContextMenu);
  if (clockTimer) {
    clearInterval(clockTimer)
  }
});

const hide = (): boolean => {
  const wasOpen = showEngines.value || isOpenSuggestions.value
  closePanels()
  return wasOpen
}

defineExpose({
  name: 'hide',
  hide
});
</script>

<style scoped>
/* ---------- 顶栏 ---------- */
.site-header {
  position: fixed;
  inset: 0 0 auto;
  z-index: 50;
  transition: background-color 0.4s ease, box-shadow 0.4s ease;
}

.site-header-scrolled {
  background: var(--glass-strong);
  -webkit-backdrop-filter: blur(24px) saturate(180%);
  backdrop-filter: blur(24px) saturate(180%);
  box-shadow: 0 1px 0 var(--hairline);
}

.brand-mark {
  display: flex;
  width: 32px;
  height: 32px;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.8);
  box-shadow: 0 2px 8px rgba(15, 23, 42, 0.08);
  transition: transform 0.35s var(--ease-apple);
}

.brand-mark img {
  width: 20px;
  height: 20px;
  object-fit: contain;
}

.dark .brand-mark {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.12);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.08);
}

.group:hover .brand-mark {
  transform: scale(1.04);
}

.header-action {
  display: inline-flex;
  width: 36px;
  height: 36px;
  align-items: center;
  justify-content: center;
  border: 0;
  border-radius: 999px;
  background: transparent;
  color: var(--ink-2);
  cursor: pointer;
  transition: background-color 0.2s ease, color 0.2s ease, transform 0.15s ease;
}

.header-action:hover {
  background: rgba(0, 0, 0, 0.055);
  color: var(--ink);
}

.dark .header-action:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.header-action:active {
  transform: scale(0.92);
}

.header-action:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.compact-search {
  display: flex;
  width: 100%;
  max-width: 420px;
  align-items: center;
  gap: 8px;
  height: 38px;
  padding: 0 6px 0 14px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.6);
  border: 1px solid var(--glass-border);
  transition: box-shadow 0.2s ease;
}

.dark .compact-search {
  background: rgba(255, 255, 255, 0.07);
}

.compact-search:focus-within {
  box-shadow: 0 0 0 4px var(--accent-soft);
}

.compact-go {
  display: flex;
  width: 28px;
  height: 28px;
  flex: none;
  align-items: center;
  justify-content: center;
  border: 0;
  border-radius: 999px;
  background: var(--accent);
  color: #fff;
  cursor: pointer;
}

.account-pop {
  position: absolute;
  right: 0;
  top: 100%;
  z-index: 80;
  padding-top: 10px;
}

/* ---------- Hero ---------- */
.hero-clock {
  font-size: clamp(56px, 9vw, 88px);
  font-weight: 200;
  line-height: 1;
  letter-spacing: -0.055em;
  font-variant-numeric: tabular-nums;
  background: linear-gradient(180deg, var(--ink) 30%, color-mix(in srgb, var(--ink) 62%, transparent));
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

/* ---------- 搜索框 ---------- */
.search-shell {
  position: relative;
  display: flex;
  height: 60px;
  align-items: center;
  gap: 10px;
  padding: 0 8px 0 12px;
  border-radius: 999px;
  background: var(--glass-strong);
  border: 1px solid var(--glass-border);
  box-shadow:
    0 1px 0 rgba(255, 255, 255, 0.7) inset,
    0 14px 44px rgba(15, 23, 42, 0.11);
  -webkit-backdrop-filter: blur(28px) saturate(180%);
  backdrop-filter: blur(28px) saturate(180%);
  transition: box-shadow 0.35s var(--ease-apple), transform 0.35s var(--ease-apple);
}

.dark .search-shell {
  box-shadow:
    0 1px 0 rgba(255, 255, 255, 0.08) inset,
    0 14px 44px rgba(0, 0, 0, 0.55);
}

.search-shell-focused {
  box-shadow:
    0 0 0 4px var(--accent-soft),
    0 18px 52px rgba(15, 23, 42, 0.15);
}

.engine-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 9px;
  border: 0;
  border-radius: 999px;
  background: transparent;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.engine-btn:hover {
  background: rgba(0, 0, 0, 0.05);
}

.dark .engine-btn:hover {
  background: rgba(255, 255, 255, 0.08);
}

.search-input {
  min-width: 0;
  flex: 1;
  border: 0;
  background: transparent;
  font-size: 15px;
  color: var(--ink);
  outline: none;
}

.search-input::placeholder {
  color: var(--ink-2);
  opacity: 0.75;
}

.search-submit {
  display: flex;
  width: 44px;
  height: 44px;
  flex: none;
  align-items: center;
  justify-content: center;
  border: 0;
  border-radius: 999px;
  background: var(--accent);
  color: #fff;
  cursor: pointer;
  box-shadow: 0 4px 14px color-mix(in srgb, var(--accent) 35%, transparent);
  transition: filter 0.2s ease, transform 0.15s ease;
}

.search-submit:hover {
  filter: brightness(1.08);
}

.search-submit:active {
  transform: scale(0.94);
}

/* ---------- 下拉面板 ---------- */
.engine-panel,
.suggest-panel {
  position: absolute;
  z-index: 70;
  padding: 6px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.99);
  border: 1px solid var(--glass-border);
  box-shadow: 0 24px 64px rgba(15, 23, 42, 0.18);
  -webkit-backdrop-filter: blur(30px) saturate(180%);
  backdrop-filter: blur(30px) saturate(180%);
}

.dark .engine-panel,
.dark .suggest-panel {
  background: rgba(30, 30, 32, 0.99);
  box-shadow: 0 24px 64px rgba(0, 0, 0, 0.7);
}

.engine-panel {
  top: calc(100% + 14px);
  left: 0;
  width: 200px;
}

.suggest-panel {
  top: calc(100% + 12px);
  left: 0;
  right: 0;
  padding: 8px;
  max-height: min(420px, 60vh);
  overflow-y: auto;
}

.panel-row {
  display: flex;
  width: 100%;
  height: 40px;
  align-items: center;
  gap: 10px;
  padding: 0 10px;
  border: 0;
  border-radius: 12px;
  background: transparent;
  color: var(--ink);
  font-size: 13.5px;
  cursor: pointer;
  transition: background-color 0.15s ease;
}

.panel-row:hover,
.panel-row-active {
  background: rgba(0, 0, 0, 0.055);
}

.dark .panel-row:hover,
.dark .panel-row-active {
  background: rgba(255, 255, 255, 0.09);
}

/* ---------- 过渡 ---------- */
.drop-enter-active,
.drop-leave-active {
  transition: opacity 0.18s ease, transform 0.22s var(--ease-apple);
}

.drop-enter-from,
.drop-leave-to {
  opacity: 0;
  transform: translateY(-4px) scale(0.98);
}

.compact-enter-active,
.compact-leave-active {
  transition: opacity 0.25s ease, transform 0.3s var(--ease-apple);
}

.compact-enter-from,
.compact-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}

@media (max-width: 640px) {
  .search-shell {
    height: 54px;
  }

  .search-submit {
    width: 40px;
    height: 40px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .site-header,
  .search-shell,
  .search-submit,
  .header-action,
  .panel-row {
    transition: none !important;
  }
}

@media (prefers-reduced-transparency: reduce) {
  .site-header-scrolled,
  .search-shell,
  .compact-search,
  .engine-panel,
  .suggest-panel {
    -webkit-backdrop-filter: none;
    backdrop-filter: none;
    background: var(--canvas);
  }
}
</style>
