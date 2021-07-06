<template>
  <div class="home">
    <div v-if="todos.length > 0">
      <v-list dark subheader :disabled="loading">
        <v-list-item
          v-for="(todo, i) in todos"
          :key="i"
          :class="{ done: todo.done }"
        >
          <template>
            <v-list-item-action>
              <v-checkbox
                :value="todo.done"
                :input-value="todo.done"
                color="teal"
                @click="toggleTodo(i)"
              ></v-checkbox>
            </v-list-item-action>

            <v-list-item-content>
              <v-list-item-title v-bind:class="{ done: todo.done }">{{
                todo.content
              }}</v-list-item-title>
            </v-list-item-content>

            <v-list-item-avatar>
              <v-btn icon @click="deleteTodo(i)">🗑️</v-btn>
            </v-list-item-avatar>
          </template>
        </v-list-item>
      </v-list>
    </div>

    <div v-else>
      <p>No todos yet!</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "TodoList",
  props: ["todos", "loading"],
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

<style>
.done {
  text-decoration: line-through;
  color: gray;
}
</style>
