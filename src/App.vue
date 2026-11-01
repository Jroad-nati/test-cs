<script setup>
import { ref, computed, onMounted } from 'vue'
const key = 'daily-todos-v1'
const todos = ref([]), text = ref(''), filter = ref('All'), loading = ref(true), adding = ref(false), error = ref(''), pending = ref(new Set())
const completed = computed(() => todos.value.filter(t => t.completed).length)
const visible = computed(() => todos.value.filter(t => filter.value === 'All' || (filter.value === 'Done' ? t.completed : !t.completed)))
const progress = computed(() => todos.value.length ? Math.round(completed.value / todos.value.length * 100) : 0)
function save() {
  try { localStorage.setItem(key, JSON.stringify(todos.value)) } catch { error.value = 'Browser storage is unavailable. Changes will last only for this visit.' }
}
async function request(path, options) {
  const response = await fetch(`https://dummyjson.com/todos${path}`, { ...options, signal: AbortSignal.timeout(15000) })
  if (!response.ok) throw new Error(`Request failed (${response.status})`)
  return response.json()
}
async function load() {
  loading.value = true; error.value = ''
  try {
    const cached = localStorage.getItem(key)
    if (cached) {
      const parsed = JSON.parse(cached)
      if (!Array.isArray(parsed) || !parsed.every(t => typeof t.todo === 'string' && typeof t.completed === 'boolean')) throw new Error('Invalid saved tasks')
      todos.value = parsed
    } else {
      const data = await request('/user/1')
      todos.value = data.todos; save()
    }
  } catch { error.value = 'Could not load your tasks. Check your connection and try again.' }
  finally { loading.value = false }
}
async function add() {
  const title = text.value.trim()
  if (!title || adding.value) return
  adding.value = true; error.value = ''
  try {
    const data = await request('/add', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ todo: title, completed: false, userId: 1 }) })
    todos.value.unshift({ ...data, id: crypto.randomUUID(), local: true, todo: title, completed: false })
    text.value = ''; save()
  } catch { error.value = 'Could not add this task. Your text is still here—please try again.' }
  finally { adding.value = false }
}
async function toggle(task) {
  if (pending.value.has(task.id)) return
  pending.value.add(task.id); error.value = ''
  try {
    // DummyJSON does not persist additions, so new tasks have no server resource to update.
    if (!task.local) await request(`/${task.id}`, { method: 'PUT', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ completed: !task.completed }) })
    task.completed = !task.completed; save()
  } catch { error.value = 'Could not update this task. Please try again.' }
  finally { pending.value.delete(task.id) }
}
onMounted(load)
</script>

<template>
  <div class="shell">
    <header><a class="brand" href="./"><span class="brand-icon">✓</span> daily<span class="brand-dot">.</span></a><span class="header-note">A LITTLE PROGRESS, EVERY DAY</span></header>
    <main>
      <div class="intro"><p class="eyebrow">MAKE ROOM FOR WHAT MATTERS</p><h1>Less on your mind.<br><span>More in your day.</span></h1><p>Your space to plan, focus, and get things done.<br>One small task at a time.</p></div>
      <section class="workspace" aria-label="Todo workspace">
        <div class="section-heading"><div><p class="eyebrow">YOUR PERSONAL CHECKLIST</p><h2>My tasks <span>{{ todos.length }}</span></h2></div><span class="today">{{ new Date().toLocaleDateString(undefined, { month: 'short', day: 'numeric' }) }}</span></div>
        <form @submit.prevent="add"><label class="sr-only" for="task">New task</label><input id="task" v-model="text" placeholder="What would you like to get done?" maxlength="300" autocomplete="off" required><button class="add" :disabled="adding || !text.trim()">{{ adding ? 'Adding…' : '+ Add task' }}</button></form>
        <div class="toolbar"><div class="tabs" aria-label="Filter tasks"><button v-for="tab in ['All', 'To do', 'Done']" :key="tab" :class="{ active: filter === tab }" :aria-pressed="filter === tab" @click="filter = tab">{{ tab }} <span>{{ tab === 'All' ? todos.length : tab === 'Done' ? completed : todos.length - completed }}</span></button></div><span class="remaining">{{ todos.length - completed }} left to do</span></div>
        <div v-if="error" class="error" role="alert">{{ error }} <button v-if="!todos.length" @click="load">Try again</button></div>
        <div v-if="loading" class="empty" role="status">Getting your day ready…</div>
        <ul v-else-if="visible.length" class="task-list"><li v-for="task in visible" :key="task.id" :class="{ done: task.completed }"><input :id="`todo-${task.id}`" type="checkbox" :checked="task.completed" :disabled="pending.has(task.id)" @change="$event.target.checked = task.completed; toggle(task)"><label :for="`todo-${task.id}`">{{ task.todo }}</label><span class="task-state">{{ pending.has(task.id) ? 'Saving…' : task.completed ? 'Completed' : 'To do' }}</span></li></ul>
        <div v-else class="empty"><span>✦</span><h3>{{ filter === 'Done' ? 'Good things take a first step.' : 'A little breathing room.' }}</h3><p>{{ filter === 'Done' ? 'Completed tasks will appear here.' : 'Add a task above, or enjoy a well-earned pause.' }}</p></div>
        <div class="progress-footer"><div><span class="progress-dot"></span>{{ completed }} of {{ todos.length }} tasks completed</div><span>{{ progress }}%</span><div class="progress-track"><div :style="{ width: progress + '%' }"></div></div></div>
      </section>
      <p class="encouragement">✧ &nbsp; Small steps. Meaningful progress.</p>
    </main>
    <footer><span>Made for a clearer day.</span><span>Powered by DummyJSON · Saved in this browser</span></footer>
  </div>
</template>
