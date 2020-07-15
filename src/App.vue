<template>
  <div id="app">
    <header>
      <h1>Millennials Chat</h1>
      <p>USER : {{ displayUsername }}</p>
    </header>

    <section>
      <div v-if="!uNameAvailable" id="change_username">
        <input id="username" v-model="usrname" type="text" v-on:keypress.enter="setUsername" />
        <button id="send_username" type="button" v-on:click="setUsername">Set username</button>
      </div>
    </section>

    <section id="chatroom">
      <div v-for="showMsg in displayMessages" v-bind:key="showMsg.index" class="Message">
        <div class="info-line"> {{ showMsg.doc.usrname }} </div>
        <div class="message"> {{ showMsg.doc.msg }} </div>
      </div>
    </section>

    <section id="input_zone">
      <input
        id="messageArea"
        v-model="msg"
        class="vertical-align"
        type="text"
        v-on:keypress.enter="sendMessage"
      />
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
      message: { _id: "", msg: "", usrname: "", date: "" },
      msg: "",
      usrname: "",
      messages: [],
      usrnameAvailable: true
    };
  },

  methods: {
    setUsername() {
      let uname = this.usrname;
      console.log("Pushing ", uname, " to _local");
      this.localstore
        .get("_local/username")
        .then(doc => {
          console.log("Getting Username : ", doc);
          doc.username = uname;
          this.localstore
            .put(doc)
            .then(() => {
              this.message.usrname = uname;
              this.usrnameAvailable = true;
            })
            .catch(console.log);
        })
        .catch(err => {
          //console.log(err);
          if (err.status == 404) {
            this.localstore
              .put({ _id: "_local/username", username: uname })
              .then(() => {
                console.log("Getting Username : ", uname);
                this.message.usrname = uname;
                this.usrnameAvailable = true;
              })
              .catch(err => {
                alert(err);
              });
          }
        });
    },
    sendMessage() {
      var sMessage = this.msg;
      this.postMessage(sMessage);
      this.msg = "";
    },
    updateStore(change) {
      try{
      console.log(" updateStore CHANGE = ", change);
      this.messages.push(change);
      console.log("AFTER PUSHING", this.messages[0].doc);
       this.messages.sort(function(a, b) {
         a = new Date(a.doc.date);
         b = new Date(b.doc.date);
         return a > b ? 1 : a < b ? -1 : 0;
       });
       console.log("After sorting, MESSAGES : ", this.messages);
      }catch(err){console.log("updateStore", err)}
    },
    postMessage: function(sMessage) {
      var timestamp = new Date().toString();
      this.message["_id"] = this.message["usrname"] + "_" + timestamp;
      this.localstore
        .put({ _id: this.message["_id"], msg: sMessage, usrname: this.message["usrname"], date: timestamp})
        .then(response => {
          console.log("Message posted");
        })
        .catch(err => {
          console.log("Message not posted", err);
        });
    },
    loadAllMessages() {
      this.localstore
        .allDocs({
          include_docs: true,
          attachments: true
        })
        .then(result => {
          console.log("RESULTS ARE :", result);
          if (result.rows) {
            this.messages = result.rows;
            this.messages.sort(function(a, b) {
              a = new Date(a.doc.date);
              b = new Date(b.doc.date);
              return a > b ? 1 : a < b ? -1 : 0;
            });
            console.log("MESSAGES : ", this.messages);
          }
        })
        .catch(err => {
          console.log(err);
        });
    },
    loadMessages() {
      console.log("CALLING loadMessages");
      this.localstore
        .changes({
          since: "now",
          live: true,
          include_docs: true
        })
        .on("change", change => {
          console.log("Changes = ",change);
          // change.id contains the doc id, change.doc contains the doc
          if (change.deleted) {
            console.log("Message deleted");
          } else {
            // document was added/modified
            this.updateStore(change);
            console.log("UPDATED");
          }
        })
        .on("error", function(err) {
          // handle errors
          alert(1);
          console.log(err);
        });
    },
    startRep() {
      PouchDB.replicate(this.localstore, this.remotestore, {
        live: true,
        retry: true
      })
        .on("change", function(info) {
          console.log("Changing: active");
        })
        .on("paused", function(err) {
          console.log("Replication: paused", err);
        })
        .on("active", function() {
          console.log("Replication: active");
        })
        .on("denied", function(err) {
          console.log("Replication: denied", err);
        })
        .on("complete", function(info) {
          console.log("Replication: complete", info);
        })
        .on("error", function(err) {
          console.log("Replication: change", err);
        });
    }
  },

  computed: {
    displayMessages() {
      return this.messages;
    },
    uNameAvailable() {
      return this.usrnameAvailable;
    },
    displayUsername() {
      return this.message.usrname;
    }
  },

  mounted() {
    this.localstore = new PouchDB("localstore");
    this.localstore
      .get("_local/username")
      .then(item => {
        console.log("USERNAME", item);
        this.message.usrname = item.username;
        this.usrnameAvailable = true;
      })
      .catch(err => {
        console.log(err);
        this.usrnameAvailable = false;
      });
    /*  this.remotestore = new PouchDB(
      "https://4f241480-c3c9-41c6-bb2e-98fd4cfe269e-bluemix:2d0f75eae437887122aec87b1225ad19a294f459beeb0a20fd69fb333cee4d4a@4f241480-c3c9-41c6-bb2e-98fd4cfe269e-bluemix.cloudantnosqldb.appdomain.cloud/simple-chat-app"
    );
    
  */ var i = 1;
     if(i){
     this.loadAllMessages();
     i=0;
     }
     this.loadMessages();
    //  this.startRep();
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
  /*  border: 5px inset #8258fa;
  position: absolute;
  margin: auto;
  height: 79%;
  width: 100%;
  overflow: auto;*/
  letter-spacing: normal;
  line-height: 21.6px;
  padding-top: 28px;
  padding-bottom: 0px;
  font-family: "Open Sans", sans-serif;
  font-weight: 400;
  font-size: 16px;
  text-rendering: auto;
  text-transform: none;
  width: 96%;
  text-indent: 0px;
  padding-left: 0px;
  padding-right: 0px;
  border-width: 0px;
  box-sizing: border-box;
}

section#input_zone {
  height: 10%;
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

input#messageArea {
  display: inline-block;
  width: 88%;
  height: 85%;
  font-family: "Comfortaa", sans-serif;
  font-size: 2vh;
}

button#send_message {
  display: inline-block;
  height: 70%;
  width: 10%;
}
/*
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
*/
.info-line {
  margin: 0 12px;
  -ms-flex: 0 1 auto;
  flex: 0 1 auto;
  padding: 0.3125rem 0;
  font-size: 12px;
  color: var(--main-gray-accessible);
}

.Message {
  display: flex;
  padding: 0.5rem 0;
  align-items: flex-end;
  padding-right: 18px;
  padding-left: 0;
  overflow: hidden;
  width: auto;
  max-width: 100%;
}

.Message .info-line {
  margin: 0 12px;
  flex: 0 1 auto;
  padding: 0.3125rem 0;
  font-size: 12px;
  color: var(--main-gray-accessible);
}

.message {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
}
</style>
