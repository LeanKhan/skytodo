<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <span v-if="loggedIn && skylink">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-chip
              v-bind="attrs"
              v-on="on"
              class="teal--text"
              :href="skylink_link"
              target="_blank"
              >{{ skylink }}</v-chip
            >
          </template>
          <span>Data Skylink</span>
        </v-tooltip>

        <v-icon class="ml-2" color="grey" small>mdi-open-in-new</v-icon>

        <!-- <v-subheader class="d-inline">skylink</v-subheader> -->
      </span>

      <v-spacer></v-spacer>

      <v-tooltip bottom>
        <template v-slot:activator="{ on, attrs }">
          <v-avatar
            v-bind="attrs"
            v-on="on"
            class="hidden-sm-and-down mr-2"
            color="light shrink"
            size="32"
          >
            {{ loggedIn ? "👽" : "🔒" }}
          </v-avatar>
        </template>
        <span> {{ loggedIn ? "Logged In" : "Not Logged in" }} </span>
      </v-tooltip>

      <v-btn class="teal--text" @click="logout" depressed small v-if="loggedIn">
        Logout
      </v-btn>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <v-container>
        <v-row>
          <v-col cols="12">
            <h3 class="text-center">To-Do</h3>
            <v-card
              class="mx-auto my-5"
              max-width="620"
              elevation="4"
              rounded
              :loading="loading"
              dark
            >
              <template slot="progress">
                <v-progress-linear
                  color="teal"
                  indeterminate
                ></v-progress-linear>
              </template>
              <v-card-text v-if="!loggedIn" class="d-flex justify-center">
                <v-btn
                  depressed
                  ripple
                  class="teal white-text"
                  :disabled="loading"
                  @click="loginWithMySky()"
                  >Login with MySky</v-btn
                ><br />
              </v-card-text>

              <div v-else>
                <v-text-field
                  color="green"
                  placeholder="I want to..."
                  style="font-size: larger"
                  hide-details
                  solo
                  flat
                  v-model="todoForm"
                  @keyup.enter="addTodo"
                >
                </v-text-field>

                <v-card-text>
                  <todo-list
                    @delete-todo="deleteTodo"
                    @toggle-todo="toggleTodo"
                    :todos="todos"
                    :loading="loading"
                  ></todo-list>
                </v-card-text>

                <v-card-actions class="p-0 justify-center">
                  <span class="grey--text caption"
                    >Saved in
                    <a
                      style="color: #00C65E"
                      href="https://blog.sia.tech/mysky-your-home-on-the-global-operating-system-of-the-future-5a288f89825c"
                      >MySky</a
                    ></span
                  >
                </v-card-actions>
              </div>
            </v-card>

            <p class="text-center">
              <img
                src="https://gblobscdn.gitbook.com/assets%2F-MQInurUBiUKRuzYEVak%2F-MYC1_8tEFTqEhnEQf_I%2F-MYC1zKJ5V5YoSVnquae%2Fbuilt%20with%20Skynet.png?alt=media&token=0b6361ab-7407-420c-850f-922d5ef253cc"
                height="30px"
              />
            </p>
            <p class="text-center text-green caption">
              by
              <a
                href="https://github.com/LeanKhan"
                class="grey--text text--darken-4"
                target="_blank"
                >@Leankhan</a
              >
            </p>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
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
    loading: false,
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
      this.loading = true;
      await this.mySky.logout();

      //set react state
      this.setLoggedIn(false);
      this.setUserID("");

      this.loading = false;
    },
    setLoggedIn(status) {
      this.loggedIn = status;
    },
    setUserID(id) {
      this.userID = id;
    },
    async saveJson() {
      this.loading = true;
      try {
        // Set discoverable JSON data at the given path. The return type is the same as getJSON.
        const { data, dataLink } = await this.mySky.setJSON(
          this.filepath,
          this.todos
        );

        console.log(data, dataLink);
        this.skylink = dataLink;
      } catch (error) {
        console.log(error);
      }
      this.loading = false;
    },
    async getJson() {
      this.loading = true;
      try {
        // Get discoverable JSON data from the given path.
        const { data, dataLink } = await this.mySky.getJSON(this.filepath);
        console.log(data, dataLink);

        // Incase it changes idk...
        this.skylink = dataLink;

        if (data) {
          this.todos = data;
        }
      } catch (error) {
        console.log(error);
      }
      this.loading = false;
    },
  },
  mounted() {
    // We'll define a portal to allow for developing on localhost.
    // When hosted on a skynet portal, SkynetClient doesn't need any arguments.
    this.loading = true;
    const portal =
      window.location.hostname === "localhost"
        ? "https://siasky.net"
        : undefined;

    // Initiate the SkynetClient
    this.client = new SkynetClient(portal);
    // just a rando path man...
    this.filepath = "localhost" + "/" + "todos.json";

    if (this.client) {
      // this.contentRecord = new ContentRecordDAC();
      this.initMySky();
    }

    this.loading = true;
  },
};
</script>
