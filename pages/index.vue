<template>
  <main class="container">
    <header class="header">
      <h1>Todo</h1>
      <p class="subtitle">Nuxt Demo (Kurt)</p>
    </header>

    <section class="card">
      <form class="row" @submit.prevent="addTodo">
        <input v-model.trim="newText" class="input" type="text" placeholder="Neues Todo …" autocomplete="off" />
        <button class="btn" type="submit" :disabled="!newText">Hinzufügen</button>
      </form>

      <div class="row row-gap">
        <label class="checkbox"><input type="checkbox" v-model="hideDone" /> Erledigte ausblenden</label>
        <button class="btn btn-ghost" type="button" @click="clearDone" :disabled="doneCount === 0">
          Erledigte löschen ({{ doneCount }})
        </button>
      </div>

      <ul class="list" v-if="filteredTodos.length">
        <li class="item" v-for="t in filteredTodos" :key="t.id">
          <label class="item-left">
            <input type="checkbox" v-model="t.done" @change="persist" />
            <span :class="['text', t.done && 'done']">{{ t.text }}</span>
          </label>
          <button class="icon" type="button" title="Löschen" @click="removeTodo(t.id)">✕</button>
        </li>
      </ul>
      <p class="empty" v-else>Keine Todos. Zeit, irgendwas zu schaffen.</p>

      <footer class="footer">
        <span>Offen: <b>{{ openCount }}</b></span>
        <span>Gesamt: <b>{{ todos.length }}</b></span>
      </footer>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

type Todo = { id: string; text: string; done: boolean }

const STORAGE_KEY = 'nuxt-todo-opencode.todos'
const todos = ref<Todo[]>([])
const newText = ref('')
const hideDone = ref(false)

const filteredTodos = computed(() => (hideDone.value ? todos.value.filter(t => !t.done) : todos.value))
const doneCount = computed(() => todos.value.filter(t => t.done).length)
const openCount = computed(() => todos.value.filter(t => !t.done).length)

function uid() {
  return `${Date.now()}-${Math.random().toString(16).slice(2)}`
}

function load() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (!raw) return
    const parsed = JSON.parse(raw)
    if (Array.isArray(parsed)) todos.value = parsed
  } catch {
    // ignore
  }
}

function persist() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
}

function addTodo() {
  if (!newText.value) return
  todos.value.unshift({ id: uid(), text: newText.value, done: false })
  newText.value = ''
  persist()
}

function removeTodo(id: string) {
  todos.value = todos.value.filter(t => t.id !== id)
  persist()
}

function clearDone() {
  todos.value = todos.value.filter(t => !t.done)
  persist()
}

onMounted(load)
</script>

<style scoped>
.container { max-width: 760px; margin: 40px auto; padding: 0 16px; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; }
.header h1 { font-size: 44px; letter-spacing: -0.02em; margin: 0; }
.subtitle { margin: 8px 0 24px; color: #6b7280; }
.card { background: white; border: 1px solid #e5e7eb; border-radius: 14px; padding: 18px; box-shadow: 0 10px 30px rgba(0,0,0,0.04); }
.row { display: flex; gap: 12px; align-items: center; }
.row-gap { margin-top: 12px; justify-content: space-between; flex-wrap: wrap; }
.input { flex: 1; border: 1px solid #e5e7eb; border-radius: 10px; padding: 10px 12px; font-size: 16px; outline: none; }
.input:focus { border-color: #93c5fd; box-shadow: 0 0 0 4px rgba(147,197,253,0.35); }
.btn { background: #111827; color: white; border: none; border-radius: 10px; padding: 10px 14px; font-weight: 600; cursor: pointer; }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-ghost { background: transparent; color: #111827; border: 1px solid #e5e7eb; }
.checkbox { display: flex; gap: 8px; align-items: center; color: #374151; }
.list { list-style: none; padding: 0; margin: 16px 0 0; }
.item { display: flex; align-items: center; justify-content: space-between; padding: 10px 6px; border-top: 1px solid #f3f4f6; }
.item-left { display: flex; gap: 10px; align-items: center; }
.text { font-size: 16px; color: #111827; }
.done { text-decoration: line-through; color: #6b7280; }
.icon { border: none; background: transparent; font-size: 18px; cursor: pointer; color: #6b7280; }
.icon:hover { color: #111827; }
.empty { margin: 18px 0 0; color: #6b7280; }
.footer { display: flex; justify-content: space-between; margin-top: 14px; padding-top: 12px; border-top: 1px solid #f3f4f6; color: #374151; }
</style>
