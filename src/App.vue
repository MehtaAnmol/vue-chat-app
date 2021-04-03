<template>
  <v-app>
    <div class="header">
      <h1>Chatroom</h1>
      <p class="username">Username:{{ username }}</p>
      <p class="online">Online: {{ users.length }}</p>
    </div>
    <chat-room :messages="messages" @sendMessage="sendMessage" />
  </v-app>
</template>

<script>
import io from "socket.io-client";
import Chatroom from "./components/Chatroom";
export default {
  name: "App",

  components: {
    "chat-room": Chatroom,
  },

  data: () => ({
    username: "",
    socket: io("http://localhost:3000"),
    messages: [],
    users: [],
  }),

  methods: {
    joinServer() {
      this.socket.on("loggedIn", (data) => {
        this.messages = data.messages;
        this.users = data.users;
        this.socket.emit("newUser", this.username);
      });

      this.listen();
    },

    listen() {
      this.socket.on("msg", (message) => {
        this.messages.push(message);
      });

      this.socket.on("userOnline", (user) => {
        this.users.push(user);
      });

      this.socket.on("userLeft", (user) => {
        this.users.splice(this.users.indexOf(user), 1);
      });
    },

    sendMessage(message) {
      this.socket.emit("msg", message);
    },
  },

  mounted() {
    this.username = prompt("What is your username", "Anonymous");

    if (!this.username) {
      this.username = "Anonymous";
    }

    this.joinServer();
  },
};
</script>

<style lang="scss">
body {
  font-family: "Avenir", Arial, Helvetica, sans-serif;
  color: #2c3e50;
  margin: 0;
  padding: 0;
}

#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100%;
  max-width: 768px;
  margin: 0 auto;
  padding: 15px;
  box-sizing: border-box;
}
</style>
