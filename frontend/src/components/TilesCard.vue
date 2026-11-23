<script setup>
import { onMounted, ref, computed } from 'vue'
import { Icon } from '@iconify/vue'
import { useSettingStore } from '../stores/SettingStore'
import AppleCard from "./AppleCard.vue";

const settingStore = useSettingStore()
const isLoading = computed(() => settingStore.isLoading)
const todos = computed(() => settingStore.getTodos)
const clipboards = computed(() => settingStore.getClipboards)
const remainingTodos = computed(() => todos.value.filter(todo => !todo.Status).length)

const newClipboard = ref('')

onMounted(() => {
  settingStore.loadUser()
})

const addClipboard = () => {
  if (!newClipboard.value.trim()) return
  settingStore.addClipboard(newClipboard.value.trim())
  newClipboard.value = ''
}
</script>

<template>
  <div v-if="isLoading" class="grid grid-cols-1 gap-4 md:grid-cols-2 md:gap-6">
    <!-- Todo 列表 -->
    <AppleCard>
      <template #title>
        <div class="flex items-center gap-3.5">
          <span class="tile-icon tile-blue">
            <Icon icon="solar:checklist-minimalistic-bold" class="h-[22px] w-[22px]" />
          </span>
          <div class="min-w-0">
            <h3 class="card-title">待办事项</h3>
            <p class="card-subtitle">
              {{ remainingTodos > 0 ? `还有 ${remainingTodos} 项未完成` : '今天的任务都完成了' }}
            </p>
          </div>
        </div>
      </template>
      <template #context>
        <ul class="mt-5 space-y-1">
          <li v-for="(todo, index) in todos" :key="index" class="list-row">
            <input
              type="checkbox"
              v-model="todo.Status"
              class="todo-check"
              @change="settingStore.saveUser()"
            />
            <span class="flex-1 truncate text-[14px]" :class="{ 'todo-done': todo.Status }">
              {{ todo.Title }}
            </span>
            <button class="row-action" aria-label="删除待办" @click="settingStore.removeTodo(index)">
              <Icon icon="solar:trash-bin-trash-linear" class="h-[18px] w-[18px]" />
            </button>
          </li>

          <li v-if="todos.length === 0" class="empty-state">
            暂无待办事项
          </li>
        </ul>
      </template>
    </AppleCard>

    <!-- 剪贴板 -->
    <AppleCard>
      <template #title>
        <div class="flex items-center gap-3.5">
          <span class="tile-icon tile-purple">
            <Icon icon="solar:clipboard-list-bold" class="h-[22px] w-[22px]" />
          </span>
          <div class="min-w-0">
            <h3 class="card-title">剪贴板</h3>
            <p class="card-subtitle">常用文本，一键复制</p>
          </div>
        </div>
      </template>
      <template #context>
        <ul class="mt-5 space-y-1">
          <li v-for="(clipboard, index) in clipboards" :key="index" class="list-row">
            <span class="flex-1 truncate text-[14px]">{{ clipboard }}</span>
            <button class="copy-btn" @click="navigator.clipboard.writeText(clipboard)">
              <Icon icon="solar:copy-linear" class="h-[15px] w-[15px]" />
              复制
            </button>
          </li>

          <li v-if="clipboards.length === 0" class="empty-state">
            暂无剪贴内容
          </li>
        </ul>

        <div class="mt-4 flex items-center gap-2">
          <input
            v-model="newClipboard"
            @keyup.enter="addClipboard"
            placeholder="添加剪贴板内容…"
            class="soft-input"
          />
          <button class="add-btn" @click="addClipboard">
            添加
          </button>
        </div>
      </template>
    </AppleCard>
  </div>
</template>

<style scoped>
.card-title {
  font-size: 19px;
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--ink);
}

.card-subtitle {
  margin-top: 2px;
  font-size: 13px;
  color: var(--ink-2);
}

.tile-icon {
  display: flex;
  width: 44px;
  height: 44px;
  flex: none;
  align-items: center;
  justify-content: center;
  border-radius: 14px;
}

.tile-blue {
  color: #0a84ff;
  background: color-mix(in srgb, #0a84ff 15%, transparent);
}

.tile-purple {
  color: #bf5af2;
  background: color-mix(in srgb, #bf5af2 15%, transparent);
}

.list-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 0 -8px;
  padding: 9px 8px;
  border-radius: 12px;
  transition: background-color 0.2s ease;
}

.list-row:hover {
  background: rgba(0, 0, 0, 0.035);
}

.dark .list-row:hover {
  background: rgba(255, 255, 255, 0.055);
}

.todo-check {
  width: 18px;
  height: 18px;
  flex: none;
  border-radius: 6px;
  accent-color: #0071e3;
  cursor: pointer;
}

.todo-done {
  color: var(--ink-2);
  text-decoration: line-through;
  opacity: 0.65;
}

.row-action {
  display: flex;
  width: 30px;
  height: 30px;
  flex: none;
  align-items: center;
  justify-content: center;
  border: 0;
  border-radius: 999px;
  background: transparent;
  color: var(--ink-2);
  cursor: pointer;
  opacity: 0;
  transition: background-color 0.2s ease, color 0.2s ease, opacity 0.2s ease;
}

.list-row:hover .row-action,
.row-action:focus-visible {
  opacity: 1;
}

.row-action:hover {
  background: rgba(255, 59, 48, 0.12);
  color: #ff3b30;
}

.copy-btn {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  flex: none;
  padding: 5px 12px;
  border: 0;
  border-radius: 999px;
  background: var(--accent-soft);
  color: var(--accent);
  font-size: 12.5px;
  font-weight: 500;
  cursor: pointer;
  transition: opacity 0.2s ease, transform 0.15s ease;
}

.copy-btn:hover {
  opacity: 0.8;
}

.copy-btn:active {
  transform: scale(0.95);
}

.empty-state {
  padding: 18px 0;
  text-align: center;
  font-size: 13px;
  color: var(--ink-2);
  opacity: 0.7;
}

.soft-input {
  min-width: 0;
  flex: 1;
  height: 38px;
  padding: 0 16px;
  border: 1px solid var(--hairline);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.5);
  color: var(--ink);
  font-size: 13.5px;
  outline: none;
  transition: box-shadow 0.2s ease, background-color 0.2s ease;
}

.dark .soft-input {
  background: rgba(255, 255, 255, 0.06);
}

.soft-input::placeholder {
  color: var(--ink-2);
}

.soft-input:focus {
  background: rgba(255, 255, 255, 0.85);
  box-shadow: 0 0 0 4px var(--accent-soft);
}

.dark .soft-input:focus {
  background: rgba(255, 255, 255, 0.1);
}

.add-btn {
  height: 38px;
  flex: none;
  padding: 0 18px;
  border: 0;
  border-radius: 999px;
  background: var(--accent);
  color: #fff;
  font-size: 13.5px;
  font-weight: 500;
  cursor: pointer;
  transition: filter 0.2s ease, transform 0.15s ease;
}

.add-btn:hover {
  filter: brightness(1.06);
}

.add-btn:active {
  transform: scale(0.96);
}

@media (hover: none) {
  .row-action {
    opacity: 1;
  }
}
</style>
