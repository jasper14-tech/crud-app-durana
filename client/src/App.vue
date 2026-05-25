<script setup>
import { ref, onMounted } from 'vue'

const API = 'http://localhost:4000/api/items'
const items = ref([])
const form = ref({ name: '', description: '', price: '' })
const editId = ref(null)

async function load() {
  items.value = await fetch(API).then(r => r.json())
}

async function save() {
  const payload = {
    ...form.value,
    price: form.value.price === '' ? null : Number(form.value.price)
  }

  if (editId.value) {
    await fetch(`${API}/${editId.value}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    })
    editId.value = null
  } else {
    await fetch(API, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    })
  }
  form.value = { name: '', description: '', price: '' }
  load()
}

function startEdit(item) {
  editId.value = item.id
  form.value = { name: item.name, description: item.description, price: item.price ?? '' }
}

async function remove(id) {
  await fetch(`${API}/${id}`, { method: 'DELETE' })
  load()
}

onMounted(load)
</script>

<template>
  <main>
    <h1>Items</h1>
    <form @submit.prevent="save">
      <input v-model="form.name" placeholder="Name" required />
      <input v-model="form.description" placeholder="Description" />
      <input v-model="form.price" type="number" min="0" step="0.01" placeholder="Price" />
      <button type="submit">{{ editId ? 'Update' : 'Add' }}</button>
    </form>
    <ul>
      <li v-for="item in items" :key="item.id">
        <strong>{{ item.name }}</strong> — {{ item.description }}
        <span v-if="item.price != null">
          — ${{ Number(item.price).toFixed(2) }}
        </span>
        <button @click="startEdit(item)">Edit</button>
        <button @click="remove(item.id)">Delete</button>
      </li>
    </ul>
  </main>
</template>