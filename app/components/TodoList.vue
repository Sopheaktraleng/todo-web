<script setup lang="ts">
import { ref, onMounted } from 'vue'
const { list, create, toggle, rename, remove } = useTasks()

type Task = { id:number; title:string; completed:boolean; createdAt:string; updatedAt:string }
const tasks = ref<Task[]>([])
const newTitle = ref('')

async function load() {
  tasks.value = await list()
}

async function addTask() {
  const title = newTitle.value.trim()
  if (!title) return
  await create(title)
  newTitle.value = ''
  await load()
}

async function toggleTask(t: Task) {
  await toggle(t.id, !t.completed)
  await load()
}

async function saveTitle(t: Task, e: Event) {
  const target = e.target as HTMLInputElement
  const title = target.value.trim()
  if (title && title !== t.title) {
    await rename(t.id, title)
    await load()
  }
}

async function delTask(id: number) {
  await remove(id)
  await load()
}

onMounted(load)
</script>

<template>
  <div class="wrapper">
    <h1>📝 Todo</h1>

    <form class="add" @submit.prevent="addTask">
      <input v-model="newTitle" placeholder="What needs to be done?" />
      <button type="submit">Add</button>
    </form>

    <ul class="list">
      <li v-for="t in tasks" :key="t.id" :class="{ done: t.completed }">
        <label>
          <input type="checkbox" :checked="t.completed" @change="toggleTask(t)" />
        </label>
        <input class="title" :value="t.title" @change="(e)=>saveTitle(t,e)" />
        <button class="del" @click="delTask(t.id)">✕</button>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.wrapper { max-width: 680px; margin: 40px auto; padding: 16px; }
h1 { margin-bottom: 12px; }
.add { display: flex; gap: 8px; margin-bottom: 16px; }
.add input { flex: 1; padding: 10px 12px; border: 1px solid #ddd; border-radius: 8px; }
.add button { padding: 10px 14px; border: 0; border-radius: 8px; cursor: pointer; }
.list { list-style: none; padding: 0; margin: 0; display: grid; gap: 8px; }
.list li { display: grid; grid-template-columns: 32px 1fr 40px; align-items: center; gap: 8px; border: 1px solid #eee; padding: 8px; border-radius: 10px; }
.list li.done .title { text-decoration: line-through; opacity: .6; }
.title { width: 100%; border: 1px solid #f1f1f1; border-radius: 8px; padding: 8px 10px; }
.del { background: transparent; border: 0; font-size: 18px; cursor: pointer; }
</style>
