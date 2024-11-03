<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const todos = ref([])
const newTodo = ref('')
const priority = ref('important-urgent')

const priorities = {
  'important-urgent': { label: '重急', color: '#ff4d4d', order: 1 },
  'important-not-urgent': { label: '重緩', color: '#ffa64d', order: 2 },
  'not-important-urgent': { label: '輕急', color: '#4da6ff', order: 3 },
  'not-important-not-urgent': { label: '輕緩', color: '#8c8c8c', order: 4 }
}

// 從 localStorage 讀取資料
onMounted(() => {
  const savedTodos = localStorage.getItem('todos')
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos)
  }
})

// 監聽 todos 變化並儲存到 localStorage
watch(todos, (newTodos) => {
  localStorage.setItem('todos', JSON.stringify(newTodos))
}, { deep: true })

// 排序後的計算屬性
const sortTodos = (todosList) => {
  return [...todosList].sort((a, b) => 
    priorities[a.priority].order - priorities[b.priority].order
  )
}

const activeTodos = computed(() => 
  sortTodos(todos.value.filter(todo => !todo.completed))
)

const completedTodos = computed(() => 
  sortTodos(todos.value.filter(todo => todo.completed))
)

const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodo.value,
      priority: priority.value,
      completed: false,
      createdAt: new Date().toISOString(),
      completedAt: null
    })
    newTodo.value = ''
  }
}

const deleteTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}

const toggleTodo = (id) => {
  const todo = todos.value.find(todo => todo.id === id)
  if (todo) {
    todo.completed = !todo.completed
    // 切換時更新完成時間
    todo.completedAt = todo.completed ? new Date().toISOString() : null
  }
}

const changePriority = (todo) => {
  const priorityKeys = Object.keys(priorities)
  const currentIndex = priorityKeys.indexOf(todo.priority)
  const nextIndex = (currentIndex + 1) % priorityKeys.length
  todo.priority = priorityKeys[nextIndex]
}

const showAllCompletedTimes = ref(false) // 新增全域顯示時間狀態

const toggleAllCompletedTimes = () => {
  showAllCompletedTimes.value = !showAllCompletedTimes.value
}

const formatDateTime = (dateString) => {
  if (!dateString) return '-'
  return new Date(dateString).toLocaleString('zh-TW', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  })
}
</script>

<template>
  <div class="todo-app">
    <h1 class="app-title">艾森豪矩陣待辦事項</h1>
    
    <div class="input-group mb-3">
      <input 
        v-model="newTodo" 
        @keyup.enter="addTodo"
        placeholder="新增待辦事項..."
        class="w-96"
      >
      <select v-model="priority" class="ms-3">
        <option 
          v-for="(info, key) in priorities" 
          :key="key" 
          :value="key"
        >
          {{ info.label }}
        </option>
      </select>
      <button @click="addTodo">新增</button>
    </div>

    <div class="tasks-container">
      <!-- 左側：進行中的任務 -->
      <div class="task-section">
        <h2 class="section-title">進行中</h2>
        <ul class="todo-list">
          <li 
            v-for="todo in activeTodos" 
            :key="todo.id"
          >
            <span 
              class="priority-tag"
              :style="{ backgroundColor: priorities[todo.priority].color }"
              @click="changePriority(todo)"
            >
              {{ priorities[todo.priority].label }}
            </span>
            <span class="todo-text">{{ todo.text }}</span>
            <div class="button-group">
              <button @click="toggleTodo(todo.id)" class="complete-btn text-base">完成</button>
              <button @click="deleteTodo(todo.id)" class="delete-btn text-base">刪除</button>
            </div>
          </li>
        </ul>
      </div>

      <!-- 右側：已完成的任務 -->
      <div class="task-section">
        <div class="section-header">
          <h2 class="section-title">已完成</h2>
          <button 
            @click="toggleAllCompletedTimes"
            class="time-btn"
          >
            {{ showAllCompletedTimes ? '隱藏完成時間' : '顯示完成時間' }}
          </button>
        </div>
        
        <ul class="todo-list todo-list-finished">
          <li 
            v-for="todo in completedTodos" 
            :key="todo.id"
            class="completed"
          >
            <div class="todo-content">
              <span 
                class="priority-tag"
                :style="{ backgroundColor: priorities[todo.priority].color }"
                @click="changePriority(todo)"
              >
                {{ priorities[todo.priority].label }}
              </span>
              <span class="todo-text">{{ todo.text }}</span>
              <div class="button-group">
                <button @click="toggleTodo(todo.id)" class="restore-btn text-base">還原</button>
                <button @click="deleteTodo(todo.id)" class="delete-btn text-base">刪除</button>
              </div>
            </div>
            <div 
              v-if="showAllCompletedTimes" 
              class="time-info"
            >
              <p>創建時間：{{ formatDateTime(todo.createdAt) }}</p>
              <p>完成時間：{{ formatDateTime(todo.completedAt) }}</p>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.todo-app {
  max-width: 1536px;
  margin: 0 auto;
  padding: 20px;
}

.app-title {
  font-size: 2.5em;
  text-align: center;
  margin-bottom: 30px;
}

.tasks-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.task-section {
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  border: 2px solid #e0e0e0;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.section-title {
  font-size: 1.8em;
  margin-bottom: 20px;
  color: #333;
}

.todo-list li {
  font-size: 1.2em;
}

.button-group {
  display: flex;
  gap: 8px;
}

.complete-btn {
  background-color: #4CAF50;
}

.restore-btn {
  background-color: #2196F3;
}

.complete-btn:hover {
  background-color: #45a049;
}

.restore-btn:hover {
  background-color: #1976D2;
}

input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  padding: 10px;
  margin-bottom: 8px;
  background-color: #f5f5f5;
  border-radius: 4px;
  gap: 10px;
  background-color: white;
  margin-bottom: 8px;
  border-radius: 4px;
}

.priority-tag {
  padding: 2px 8px;
  border-radius: 12px;
  color: white;
  font-size: 0.8em;
  cursor: pointer;
  transition: transform 0.2s;
}

.priority-tag:hover {
  transform: scale(1.05);
  opacity: 0.9;
}

.todo-text {
  flex: 1;
}

.completed .todo-text {
  color: #888;
}

.delete-btn {
  background-color: #ff4444;
}

.delete-btn:hover {
  background-color: #cc0000;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.todo-list-finished li {
  display: flex;
  flex-direction: column; /* 改為垂直排列 */
  width: 100%;
  overflow: hidden;
}

.todo-content {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 12px;
}

.time-info {
  width: 100%;
  padding: 8px 12px;
  background-color: #f8f9fa;
  border-top: 1px solid #eee;
  border-left: 6px solid #9c27b0;
  font-size: 0.9em;
  color: #666;
}

.time-btn {
  background-color: #9c27b0;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s;
}

.time-btn:hover {
  background-color: #7b1fa2;
}
</style>