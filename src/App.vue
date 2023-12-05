<script>
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
  methods: {
    onSubmit() {
      if (this.name.length < 1) return;
      let todo = {
        todo: this.name,
        completed: false,
      };
      this.todos.push(todo);
      this.name = '';
    },
    updateSelectedId(index) {
      if (this.name) return;
      if (this.selectedId !== null) return;
      this.selectedId = index;
    },
    updateTodos() {
      let newTodos = this.todos.map((item, index) =>
        index === this.selectedId
          ? { todo: this.editedTodo, completed: false }
          : item
      );
      this.todos = newTodos;

      this.selectedId = null;
      this.editedTodo = null;
    },
    cancelUpdate() {
      this.selectedId = null;
      this.editedTodo = null;
    },
    deleteTodo(index) {
      let newTodos = this.todos.filter((todo, id) => id !== index);
      this.todos = newTodos;
    },
    isCompleted(index) {
      if (this.selectedId !== null) return;
      let newTodos = this.todos.map((item, id) =>
        id === index ? { ...item, completed: !item.completed } : item
      );
      this.todos = newTodos;
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

      <div>
        <div
          v-show="todos.length < 1"
          class="text-center text-xl my-8 py-8 text-slate-200 bg-slate-800"
        >
          You have no Todos in your list 😟
        </div>

        <ul v-show="todos.length >= 1" class="bg-slate-800 text-slate-200 mt-6">
          <p class="py-8 text-xl text-center" v-show="displayedValue < 1">
            You don't have any {{ this.category }} todos
            {{ this.category === 'completed' ? '😟' : '💪' }}
          </p>
          <li v-for="(todo, index) in displayedValue" :key="index">
            <div
              v-if="selectedId !== index"
              class="flex border-b border-slate-100 items-center gap-4 pl-3"
            >
              <div>
                <input
                  v-if="!todo.completed"
                  :checked="todo.completed"
                  class="appearance-none bg-slate-800 border border-slate-200 w-6 h-6 rounded-full checked:bg-blue-200 cursor-pointer"
                  type="radio"
                  @click="isCompleted(index)"
                />
                <div
                  v-else
                  class="w-6 h-6 rounded-full ticked border border-slate-200 flex items-center justify-center"
                >
                  <img
                    src="./assets/images/icon-check.svg"
                    alt="okay"
                    @click="isCompleted(index)"
                  />
                </div>
              </div>

              <p
                :class="{ 'line-through': todo.completed }"
                class="capitalize cursor-pointer flex-1 py-3"
                @click="isCompleted(index)"
              >
                {{ todo.todo }}
              </p>
              <div class="ml-auto flex gap-2">
                <button
                  class="px-3 py-2"
                  @click="updateSelectedId(index, todo.todo)"
                  v-show="!todo.completed"
                >
                  ✏️
                </button>
                <button
                  :disabled="selectedId !== null"
                  class="px-3 py-2 disabled:cursor-not-allowed justify-self-end"
                  @click="deleteTodo(index)"
                >
                  ✖️
                </button>
              </div>
            </div>
            <form
              v-else
              class="flex items-center gap-4 update"
              @submit.prevent="updateTodos"
            >
              <input
                type="text"
                class="update-input outline-none bg-slate-600 text-slate-200 flex-1 px-3 py-3"
                v-model="editedTodo"
              />
              <button class="ml-auto px-3 py-2">✔️</button>

              <button type="button" class="px-3 py-2" @click="cancelUpdate">
                ⭕
              </button>
            </form>
          </li>
        </ul>
        <div
          v-show="todos.length >= 1"
          class="flex justify-between items-center text-slate-300 gap-4 bg-slate-800 px-3 py-4"
        >
          <p>
            {{ itemsLeft || 'No' }}
            {{ itemsLeft > 1 ? 'items' : 'item' }} Left
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
