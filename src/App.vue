<template>
  <div id="app" class="container">
    <header>
      <h1>Sky-Todos!</h1>
      <span style="float: right">
        {{ loggedIn ? "Logged In" : "Not logged in" }}
      </span>
    </header>
    <br />

    <div v-if="!loggedIn">
      <button @click="loginWithMySky()">Login with MySky</button><br />
    </div>

    <div class="containter" v-else>
      <input type="text" v-model="todoForm" @keyup.enter="addTodo" />

      <todo-list
        @delete-todo="deleteTodo"
        @toggle-todo="toggleTodo"
        :todos="todos"
      ></todo-list>
    </div>
  </div>
</template>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
<script>
import TodoList from "./components/Todos.vue";
// Import the SkynetClient and a helper
import { SkynetClient } from "skynet-js";
// import { ContentRecordDAC } from '@skynetlabs/content-record-library';

export default {
  components: {
    TodoList,
  },
  data: () => ({
    todoForm: "",
    client: null,
    mySky: null,
    loggedIn: false,
    file: null,
    skylink: "",
    userID: "",
    filepath: "",
    todos: [],
  }),
  computed: {
    skylink_link() {
      return "https://siasky.net/" + this.skylink.replace("sia:", "") + "/";
    },
  },
  methods: {
    addTodo() {
      if (this.todoForm) {
        this.todos.push({ content: this.todoForm, done: false });
      }

      this.todoForm = "";

      this.saveJson();
    },
    deleteTodo(todo_id) {
      this.todos.splice(todo_id, 1);

      this.saveJson();
    },
    toggleTodo(todo_id) {
      this.todos[todo_id].done = !this.todos[todo_id].done;

      this.saveJson();
    },
    // define async setup function
    async initMySky() {
      try {
        // load invisible iframe and define app's data domain
        // needed for permissions write
        const datadomain = "localhost";
        const mySky = await this.client.loadMySky(datadomain, {
          dev: true,
          debug: true,
        });

        // load necessary DACs and permissions
        // await this.mySky.loadDacs(this.contentRecord);

        // check if user is already logged in with permissions
        const loggedIn = await mySky.checkLogin();

        // set react state for login status and
        // to access mySky in rest of app
        this.mySky = mySky;

        this.setLoggedIn(loggedIn);

        if (loggedIn) {
          this.setUserID(await mySky.userID());
        }

        // after initializing MySky
        this.getJson();

        // next, check if there is user todos saved!
      } catch (e) {
        console.error(e);
      }
    },
    // call async setup function
    async loginWithMySky() {
      // Try login again, opening pop-up. Returns true if successful
      const status = await this.mySky.requestLoginAccess();

      // set react state
      this.setLoggedIn(status);

      if (status) {
        this.setUserID(await this.mySky.userID());
      }
    },
    async logout() {
      // call logout to globally logout of mysky
      await this.mySky.logout();

      //set react state
      this.setLoggedIn(false);
      this.setUserID("");
    },
    setLoggedIn(status) {
      this.loggedIn = status;
    },
    setUserID(id) {
      this.userID = id;
    },
    async saveJson() {
      try {
        // Set discoverable JSON data at the given path. The return type is the same as getJSON.
        const { data, dataLink } = await this.mySky.setJSON(
          this.filepath,
          this.todos
        );

        console.log(data, dataLink);
      } catch (error) {
        console.log(error);
      }
    },
    async getJson() {
      try {
        // Get discoverable JSON data from the given path.
        const { data, dataLink } = await this.mySky.getJSON(this.filepath);
        console.log(data, dataLink);

        if (data) {
          this.todos = data;
        }
      } catch (error) {
        console.log(error);
      }
    },
  },
  mounted() {
    // We'll define a portal to allow for developing on localhost.
    // When hosted on a skynet portal, SkynetClient doesn't need any arguments.
    const portal =
      window.location.hostname === "localhost"
        ? "https://siasky.net"
        : undefined;

    // Initiate the SkynetClient
    this.client = new SkynetClient(portal);
    this.filepath = "localhost" + "/" + "todos.json";

    if (this.client) {
      // this.contentRecord = new ContentRecordDAC();
      this.initMySky();
    }
  },
};
</script>
