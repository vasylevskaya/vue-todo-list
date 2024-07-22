<script>
  import todos from "./data/todos";
  import StatusFilter from './components/StatusFilter.vue';
  import TodoItem from './components/TodoItem.vue';
  import Message from './components/Message.vue';
  import { createTodo, getTodos, updateTodo, deleteTodo } from "./api/todos";

  export default {
    components: {
      Filter: StatusFilter,
      TodoItem,
      Message,
    },
    data() {
      return {
        todos: [],
        title: '',
        status: 'all'
      }
    },
    computed: {
      activeTodos() {
        return this.todos.filter(todo => !todo.completed)
      },
      completedTodos() {
        return this.todos.filter(todo => todo.completed)
      },
      visibleTodos() {
        switch (this.status) {
          case 'active':
            return this.activeTodos;

          case 'completed':
            return this.completedTodos;

          default:
            return this.todos;
        }
      }
    },
    /* watch: {
      todos: {
        deep: true,
        handler() {
          localStorage.setItem('todos', JSON.stringify(this.todos))
        }
      }
    }, */
    mounted() {
      getTodos()
      .then(({ data }) => {
        this.todos = data
      })
      .catch(() => {
        this.$refs.errorMessage.show('Unable to load todos')
      })
    },
    updated() {
    },
    methods: {
      handleSubmit() {
        createTodo(this.title)
        .then(({ data }) => {
          
          this.todos == [ ...this.todos, data ]
          this.title = ''
        })
        .catch(() => {
          this.$refs.errorMessage.show('Unable to create todo')
        })
      },
      updateTodo({ id, title, completed }) {
        updateTodo({ id, title, completed })
          .then(({ data }) => {
            this.todos = this.todos.map(
              todo => todo.id !== id ? todo : data,
            )
          })
          .catch(() => {
            this.$refs.errorMessage.show('Unable to update todo')
          })
      },
      deleteTodo(todoId) {
        deleteTodo(todoId)
          .then(() => {
            this.todos = this.todos.filter(
              todo => todo.id !== todoId,
            )
          })
          .catch(() => {
            this.$refs.errorMessage.show('Unable to delete todo')
          })
      }
    }
}
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">
      todos
    </h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
        ></button>
        <form @submit.prevent="handleSubmit">
          <input
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
          />
        </form>
      </header>

      <TransitionGroup
        name="list"
        tag="section"
        class="todoapp__main"
      >
        <TodoItem
          v-for="(todo, index) in visibleTodos"
          :key="todo.id"
          :todo="todo"
          @update="updateTodo"
          @delete="deleteTodo(todo.id)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer">
        <span class="todo-count">
          {{ activeTodos.length }} items left
        </span>

        <Filter
          v-model="status"
        />

        <button class="todoapp__clear-completed" v-if="completedTodos.length > 0" @click="todos = activeTodos">
          Clear completed
        </button>
      </footer>
    </div>

    <!--
      With ref we get access to "this" of child component.
      Be careful and use refs exceptionally, because it breaks
      parent-child chain
    -->
    <Message
      class="is-warning"
      ref="errorMessage"
    >
      <template #default="{ text }">
        <p>{{ text }}</p>
      </template>
      
      <template #header>
        <p>Server Error</p>
      </template>
    </Message>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
