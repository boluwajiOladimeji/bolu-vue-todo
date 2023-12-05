<script>
import { v4 as uuidv4 } from 'uuid';
export default {
  data() {
    return {
      name: '',
      todos: [],
      selectedId: null,
      editedTodo: '',
      category: 'all',
    };
  },

  created() {
    this.loadTodos();
  },

  methods: {
    loadTodos() {
      const loadedTodos = localStorage.getItem('todos');
      if (loadedTodos) {
        this.todos = JSON.parse(loadedTodos);
      }
    },
    saveTodos() {
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },

    onSubmit() {
      if (this.name.length < 1) return;
      let todo = {
        id: uuidv4(),
        todo: this.name,
        completed: false,
      };
      this.todos.push(todo);
      this.name = '';
      this.category = 'all';
    },
    updateSelectedId(index, todo) {
      this.editedTodo = todo;
      if (this.name) return;
      if (this.selectedId !== null) return;
      this.selectedId = index;
    },
    updateTodos(todo) {
      if (todo === '') return;
      let newTodos = this.todos.map((item, index) =>
        index === this.selectedId ? { todo: todo, completed: false } : item
      );
      this.todos = newTodos;

      this.selectedId = null;
      this.editedTodo = null;
    },
    cancelUpdate(index) {
      this.selectedId = null;
      const newTodos = this.todos.map((todo, id) =>
        id === index ? { todo: this.editedTodo, completed: false } : todo
      );
      this.todos = newTodos;
    },
    deleteTodo(index) {
      let newTodos = this.todos.filter((todo, id) => id !== index);
      this.todos = newTodos;
    },
    isCompleted(id) {
      if (this.selectedId !== null) return;
      let newTodos = this.todos.map((item) =>
        id === item.id ? { ...item, completed: !item.completed } : item
      );
      this.todos = newTodos;
      // console.log(todo);
    },
    setCategory(category) {
      this.category = category;
    },
    clearAll() {
      this.name = '';
      this.todos = [];
      this.editedTodo = '';
      this.selectedId = null;
      this.category = 'all';
    },
  },

  watch: {
    todos: {
      handler() {
        this.saveTodos();
      },
      deep: true,
    },
  },

  computed: {
    displayedValue() {
      if (this.category === 'all') {
        return this.todos;
      }

      if (this.category === 'completed') {
        return this.todos.filter((todo) => todo.completed);
      }

      if (this.category === 'uncompleted') {
        return this.todos.filter((todo) => !todo.completed);
      }
    },
    itemsLeft() {
      return this.todos.filter((todo) => !todo.completed).length;
    },
  },
};
</script>

<template>
  <main class="section-center min-h-screen py-16 w-full">
    <div class="w-11/12 max-w-lg mx-auto">
      <h1 class="text-3xl tracking-wider mb-8 text-slate-200">TODO</h1>

      <form
        class="form flex text-center w-full text-slate-200"
        @submit.prevent="onSubmit"
      >
        <input
          :disabled="selectedId !== null"
          v-model="name"
          class="todo-input py-2 px-3 w-3/4 outline-none disabled:cursor-not-allowed"
          type="text"
        />
        <button
          :disabled="selectedId !== null"
          class="todo-btn flex-1 text-center text-slate-200 border-l border-slate-200 disabled:cursor-not-allowed"
        >
          Add
        </button>
      </form>

      <div class="min-h-[200px] bg-slate-800 flex flex-col mt-8">
        <div
          v-show="todos.length < 1"
          class="text-center text-xl my-8 py-8 text-slate-200 bg-slate-800"
        >
          You have no tasks to do 📕
        </div>

        <ul v-show="todos.length >= 1" class="bg-slate-800 text-slate-200">
          <p
            class="py-8 text-center px-3 text-sm md:text-xl"
            v-show="displayedValue < 1"
          >
            You don't have any {{ this.category }} task
            {{ this.category === 'completed' ? '😟' : '💪' }}
          </p>
          <li v-for="(todo, index) in displayedValue" :key="index">
            <div
              v-if="selectedId !== index"
              class="flex border-b border-slate-100 items-center gap-4 px-6 text-sm"
            >
              <div>
                <input
                  v-if="!todo.completed"
                  :checked="todo.completed"
                  class="appearance-none bg-slate-800 border border-slate-200 w-5 h-5 rounded-full cursor-pointer"
                  type="radio"
                  @click="isCompleted(todo.id)"
                />
                <div
                  v-else
                  class="w-5 h-5 rounded-full ticked border border-slate-200 flex items-center justify-center"
                >
                  <img
                    src="./assets/images/icon-check.svg"
                    alt="okay"
                    @click="isCompleted(todo.id)"
                  />
                </div>
              </div>

              <p
                :class="{ 'line-through': todo.completed }"
                class="capitalize cursor-pointer py-3 flex-1 max-w-sm"
                @click="isCompleted(todo.id)"
              >
                {{ todo.todo }}
              </p>
              <div class="ml-auto flex gap-8">
                <button
                  @click="updateSelectedId(index, todo.todo)"
                  v-show="!todo.completed"
                >
                  ✏️
                </button>
                <button
                  :disabled="selectedId !== null"
                  class="disabled:cursor-not-allowed justify-self-end"
                  @click="deleteTodo(index)"
                >
                  ✖️
                </button>
              </div>
            </div>
            <form
              v-else
              class="flex items-center gap-4 update"
              @submit.prevent="updateTodos(todo.todo)"
            >
              <input
                type="text"
                class="update-input outline-none bg-slate-600 text-slate-200 flex-1 px-3 py-3"
                v-model="todo.todo"
              />
              <button class="ml-auto px-3 py-2">✔️</button>

              <button
                type="button"
                class="px-3 py-2"
                @click="cancelUpdate(index)"
              >
                ⭕
              </button>
            </form>
          </li>
        </ul>
        <div
          v-show="todos.length >= 1"
          class="flex justify-between items-center text-slate-300 gap-4 text-xs md:text-base bg-slate-800 px-3 py-4 mt-auto"
        >
          <p>
            {{ itemsLeft || 'No' }}
            {{ itemsLeft > 1 ? 'Tasks' : 'Task' }} Left
          </p>
          <div class="flex gap-3">
            <button
              class="category"
              :class="{ 'text-blue-500': category === 'all' }"
              @click="setCategory('all')"
            >
              All
            </button>
            <button
              class="category"
              :class="{ 'text-blue-500': category === 'uncompleted' }"
              @click="setCategory('uncompleted')"
            >
              Uncompleted
            </button>
            <button
              class="category"
              :class="{ 'text-blue-500': category === 'completed' }"
              @click="setCategory('completed')"
            >
              Completed
            </button>
          </div>
          <button @click="clearAll">Clear All</button>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped></style>
