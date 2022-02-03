<template>
  <div
      id="app"
      class="container"
  >
    <form
        class="row"
        @submit.prevent="send"
        id="Chat"
    >
      <input
          type="text"
          v-model="username"
          placeholder="digite o usuario"
      />
      <div
          id="tela"
          class="messages"
      >
        <div
            v-for="men in messages"
            :key="men.username"
        >
          <strong>{{ men.username }}</strong> : {{ men.newMessage }}
        </div>
      </div>
      <input
          type="text"
          v-model="newMessage"
          placeholder="digite a menssagem"
      />
      <button type="submit">Enviar</button>
    </form>
  </div>

</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";

export default {
  name: "App",
  components: {},
  data() {
    return {
      messages: [],
      newMessage: null,
      username: null,
      socket: null
    };
  },
  methods: {
    send() {
      console.log("Send message:" + this.newMessage);
      if (this.stompClient && this.stompClient.connected) {
        var msg = {newMessage: this.newMessage, username: this.username};
        this.stompClient.send("/app/chat.sendMessage", JSON.stringify(msg), {});
      }
    },
    connect() {
      this.socket = new SockJS("http://localhost:8081/ws");
      this.stompClient = Stomp.over(this.socket);
      this.stompClient.connect(
          {},
          frame => {
            this.connected = true;
            console.log(frame);
            this.stompClient.subscribe("/topic/public", tick => {
              console.log("payload", JSON.parse(tick.body));
              this.messages.push(JSON.parse(tick.body));
            });
          },
          error => {
            console.log(error);
            this.connected = false;
          }
      );
    },
    disconnect() {
      if (this.stompClient) {
        this.stompClient.disconnect();
      }
      this.connected = false;
    },
    tickleConnection() {
      this.connected ? this.disconnect() : this.connect();
    }
  },
  mounted() {
    this.connect();
  }
};
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: 50px;
  margin-right: 800px;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#chat {
  height: 80%;
  display: inline;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

input {
  width: 600px;
  border: 1px solid #ddd;
  height: 50px;
  padding: 0 20px;
  font-size: 14px;
}

button {
  width: 600px;
  height: 50px;
  font-size: 14px;
  background: #069;
  text-align: center;
  line-height: 50px;
  font-weight: bold;
  color: #fff;
  margin-top: 10px;
}

.messages {
  width: 600px;
  height: 400px;
  margin: 20px 0;
  border: 1px solid #ddd;
  padding: 20px;
}
</style>
