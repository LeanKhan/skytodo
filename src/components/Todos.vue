<template>
  <div class="home">
    <div v-if="todos.length > 0">
      <h1>Todos</h1>
      <p>
        Completed:
        {{
          todos.filter((todo) => {
            return todo.done;
          }).length
        }}
      </p>

      <ul>
        <li v-for="(todo, i) in todos" :key="i" :class="{ done: todo.done }">
          <div>
            <input
              type="checkbox"
              :checked="todo.done"
              @click="toggleTodo(i)"
            />

            {{ todo.content }}

            <button @click="deleteTodo(i)">x</button>
          </div>
        </li>
      </ul>
    </div>

    <div v-else>
      <p>No todos yet!</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "TodoList",
  props: ["todos"],
  methods: {
    deleteTodo(todo) {
      this.$emit("delete-todo", todo);
    },
    toggleTodo(todo) {
      this.$emit("toggle-todo", todo);
    },
  },
};
</script>
