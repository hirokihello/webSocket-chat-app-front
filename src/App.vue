<template>
    <main id="app">
        <div class="row">
            <div class="col s12">
                <div class="card horizontal">
                    <div id="chat-messages" class="card-content" v-html="chatContent">
                    </div>
                </div>
            </div>
        </div>
        <div class="row" v-if="joined">
            <div class="input-field col s8">
                <input type="text" v-model="newMsg" @keyup.enter="send">
            </div>
            <div class="input-field col s4">
                <button class="waves-effect waves-light btn" @click="send">
                    <i class="material-icons right">chat</i>
                    Send
                </button>
            </div>
        </div>
        <div class="row" v-if="!joined">
            <div class="input-field col s8">
                <input type="email" v-model.trim="email" placeholder="Email">
            </div>
            <div class="input-field col s8">
                <input type="text" v-model.trim="username" placeholder="Username">
            </div>
            <div class="input-field col s4">
                <button class="waves-effect waves-light btn" @click="join()">
                    <i class="material-icons right">done</i>
                    Join
                </button>
            </div>
        </div>
    </main>
</template>

<script>
import Materialize from "materialize-css"
import md5 from 'crypto-js/md5';
export default {
  name: 'App',
  data() {
    return {
      ws: null,
      newMsg: '',
      chatContent: '',
      email: null,
      username: null,
      joined: false
    }
  },
  created() {
      var self = this;
      this.ws = new WebSocket('ws://' + 'localhost:8000' + '/ws');
      this.ws.addEventListener('message', function(args) {
          const msg = JSON.parse(args.data);
          self.chatContent += '<div class="chip">'
              + '<img src="' + self.gravatarURL(msg.email) + '">'
              + msg.username
              + '</div>'
              + '<p>' + msg.message + '</p>' + '<br/>';

          const element = document.getElementById('chat-messages');
          element.scrollTop = element.scrollHeight;
      });
  },
  methods: {
    send() {
      if (this.newMsg != '') {
        this.ws.send(
          JSON.stringify({
              email: this.email,
              username: this.username,
              message: this.newMsg
            }
          ));
        this.newMsg = '';
      }
    },
    join() {
      if (!this.email) {
        Materialize.toast('You must enter an email', 2000);
        return
      }
      if (!this.username) {
        Materialize.toast('You must choose a username', 2000);
        return
      }
      this.joined = true;
    },
    gravatarURL(email) {
      return 'http://www.gravatar.com/avatar/' + md5(email);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
