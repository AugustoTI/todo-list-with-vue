<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue'

interface Todo {
  created_at: number
  content: string
  category: string
  done: boolean
}

const todos = ref<Todo[]>([])
const name = ref('')
const input_content = ref('')
const input_category = ref<string | null>('')

const todos_asc = computed(() =>
  // eslint-disable-next-line vue/no-side-effects-in-computed-properties
  todos.value.sort((a, b) => {
    return b.created_at - a.created_at
  }),
)

const addTodo = () => {
  if (!input_content.value.trim() || !input_category.value) return

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    created_at: new Date().getTime(),
    done: false,
  })

  input_content.value = ''
  input_category.value = null
}

const removeTodo = (todo: Todo) => {
  todos.value = todos.value.filter((t) => t !== todo)
}

onMounted(() => {
  name.value = localStorage.getItem('name') || ''
  const todosRaw = localStorage.getItem('todos')
  const todosJSON = todosRaw ? JSON.parse(todosRaw) : []
  todos.value = todosJSON
})

watch(name, (newValue) => {
  localStorage.setItem('name', newValue)
})

watch(
  todos,
  (newValue) => {
    localStorage.setItem('todos', JSON.stringify(newValue))
  },
  { deep: true },
)
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up, <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>
      <form @submit.prevent="addTodo">
        <div>
          <label for="content" class="label">What's on your todo list?</label>
          <input
            id="content"
            type="text"
            placeholder="e.g make a video"
            v-model="input_content"
          />
        </div>

        <div>
          <h4>Pick a category</h4>
          <div class="options">
            <label>
              <input
                type="radio"
                name="category"
                value="business"
                v-model="input_category"
              />
              <span class="bubble business"></span>
              <div>Business</div>
            </label>
            <label>
              <input
                type="radio"
                name="category"
                value="personal"
                v-model="input_category"
              />
              <span class="bubble personal"></span>
              <div>Personal</div>
            </label>
          </div>
        </div>

        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <ul class="list">
        <li
          v-for="todo in todos_asc"
          v-bind:key="todo.created_at"
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="delete" type="button" @click="removeTodo(todo)">Delete</button>
          </div>
        </li>
      </ul>
    </section>
  </main>
</template>
