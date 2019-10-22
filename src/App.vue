<template>
  <div id="app">
    <header>
      <h1>Super Chat</h1>
    </header>

    <section>
      <div id="change_username">
        <input id="username" v-model="usrname" type="text" />
        <button id="send_username" type="button" v-on:click="setUsername">Change username</button>
      </div>
    </section>

    <section id="chatroom">
      
    </section>

    <section id="input_zone">
      <input id="message" v-model="msg" class="vertical-align" type="text"/>
      <button id="send_message" class="vertical-align" v-on:click="sendMessage" type="button">Send</button>
    </section>
  </div>
</template>

<script>
var PouchDB = require("pouchdb").default;

export default {
  data() {
    return {
      localstore: {},
      message: {msg:'', usrname:''},
      msg: '',
      usrname:'',
      messages:[]
    };
  },

  methods: {
    setUsername() {
      this.message.usrname = this.usrname;
      this.usrname = "";
    },
    sendMessage () {
      this.message.msg = this.msg;
      this.postMessage();
      this.msg = "";
    },
    postMessage: function() {
      this.localstore
        .post({
          username: this.message.usrname,
          message: this.message.msg,
          time: new Date()
        })
        .then(response => {
          console.log("Message posted")
        })
        .catch(err => {
          console.log("Message not posted", err);
        });
    },
    loadMessages() {
      this.localstore
        .allDocs({
          include_docs: true,
          attachments: true
        })
        .then(result => {
          console.log("RESULTS ARE :", result)
        })
        .catch(err => {
          console.log(err);
        });
    },

  },

  computed: {

  },

  mounted() {
    this.localstore = new PouchDB("localstore");

  }
};
</script>



<style scoped lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: "100%";
  width: "100%";
}

@mixin vertical-align {
  position: relative;
  top: 50%;
  -webkit-transform: translateY(-50%);
  -ms-transform: translateY(-50%);
  transform: translateY(-50%);
}

* {
  box-sizing: border-box;
}

html {
  height: 100%;
}

body {
  margin: 0px;
  height: 100%;
}

header {
  text-align: center;
}

h1 {
  font-family: "Comfortaa", sans-serif;
  margin: 0px;
}

div#change_username,
section#input_zone {
  border: 5px outset #8258fa;
}

div#change_username {
  height: 3%;
  display: inline-block;
  background-color: #bca9f5;
}

section#chatroom {
  border: 5px inset #8258fa;
  position: absolute;
  margin: auto;
  height: 85%;
  width: 100%;
  overflow: auto;
  bottom: 0px;
}

section#input_zone {
  height: 7%;
  text-align: center;
  background-color: #bca9f5;
  position: fixed;
  bottom: 0;
  width: 100%;
}

.vertical-align {
  position: relative;
  top: 50%;
  -webkit-transform: translateY(-50%);
  -ms-transform: translateY(-50%);
  transform: translateY(-50%);
}

input#username {
  font-family: "Comfortaa", sans-serif;
}

input#message {
  display: inline-block;
  width: 88%;
  height: 70%;
  font-family: "Comfortaa", sans-serif;
  font-size: 2vh;
}

button#send_message {
  display: inline-block;
  height: 70%;
  width: 10%;
}

p.message {
  margin: 0px;
  width: 100%;
  height: 40px;
  font-size: 2vh;
  font-family: "Comfortaa", sans-serif;
  padding: 1vh;
}

.message:nth-child(even) {
  background-color: #81daf5;
}

.message:nth-child(odd) {
  background-color: #81bef7;
}
</style>
