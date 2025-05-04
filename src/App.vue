<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const newTask = ref('')
const selectedCategory = ref(null)
const filterGender = ref('all') // 'all', 'male', 'female'

// Filter todos based on selected gender
const filteredTodos = computed(() => {
  if (filterGender.value === 'all') return todos.value
  return todos.value.filter(todo => todo.category === filterGender.value)
})

const addTodo = () => {
  if (newTask.value.trim() === '' || !selectedCategory.value) return
  
  todos.value.unshift({
    id: Date.now(),
    task: newTask.value,
    category: selectedCategory.value,
    done: false,
    createdAt: new Date().getTime()
  })
  
  newTask.value = ''
  selectedCategory.value = null
}

const removeTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}

const toggleDone = (id) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}

// Load saved todos
onMounted(() => {
  const saved = localStorage.getItem('todos')
  if (saved) todos.value = JSON.parse(saved)
  
  // Auto-save
  watch(todos, (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal))
  }, { deep: true })
})
</script>

<template>
  <div class="background-animation"></div>
  <div class="todo-app">
    <header class="app-header">
      <h1>Nama Peserta Ujian</h1>
      <div class="stats">
        <div class="stat">
          <span class="count">{{ todos.length }}</span>
          <span class="label">Total</span>
        </div>
        <div class="stat">
          <span class="count">{{ todos.filter(t => t.done).length }}</span>
          <span class="label">Selesai</span>
        </div>
      </div>
    </header>
    
    <!-- Input Section -->
    <div class="input-section">
      <input 
        type="text" 
        v-model="newTask"
        placeholder="Masukan Nama"
        @keyup.enter="addTodo"
        class="task-input"
      >
      
      <div class="categories">
        <button 
          @click="selectedCategory = 'male'"
          :class="{ active: selectedCategory === 'male' }"
          class="male-btn"
        >
          <i class="icon">♂</i> Laki-laki
        </button>
        <button 
          @click="selectedCategory = 'female'"
          :class="{ active: selectedCategory === 'female' }"
          class="female-btn"
        >
          <i class="icon">♀</i> Perempuan
        </button>
      </div>
      
      <button @click="addTodo" class="add-btn">
        <i class="fas fa-plus"></i> Tambah Nama
      </button>
    </div>
    
    <!-- Filter Section -->
    <div class="filter-section">
      <button 
        @click="filterGender = 'all'"
        :class="{ active: filterGender === 'all' }"
        class="filter-btn"
      >
        Semua
      </button>
      <button 
        @click="filterGender = 'male'"
        :class="{ active: filterGender === 'male' }"
        class="filter-btn male-filter"
      >
        Laki-laki
      </button>
      <button 
        @click="filterGender = 'female'"
        :class="{ active: filterGender === 'female' }"
        class="filter-btn female-filter"
      >
        Perempuan
      </button>
    </div>
    
    <!-- Todo List -->
    <div class="todo-list">
      <div 
        v-for="todo in filteredTodos" 
        :key="todo.id"
        class="todo-item"
        :class="{ done: todo.done, male: todo.category === 'male', female: todo.category === 'female' }"
      >
        <div class="checkbox" @click="toggleDone(todo.id)">
          <span v-if="todo.done">✓</span>
        </div>
        
        <div class="task-content">
          <span class="task">{{ todo.task }}</span>
          <span class="category-badge" :class="todo.category">
            {{ todo.category === 'male' ? '♂' : '♀' }}
          </span>
        </div>
        
        <button @click="removeTodo(todo.id)" class="delete-btn">
          <i class="fas fa-trash"></i>
        </button>
      </div>
      
      <div v-if="filteredTodos.length === 0" class="empty-state">
        <i class="fas fa-clipboard-list"></i>
        <p>Tidak ada nama peserta</p>
      </div>
    </div>
  </div>
</template>