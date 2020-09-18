<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
          Chat
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-content>
          <v-container class="fill-height">
              <v-row class="fill-height pb-14" align="start">
                <v-col>
                    <v-alert type="error" v-if="!isValid" transition="scale-transition">
                        {{infoMsg}}
                     </v-alert>
                      <ul ref="chatbox">
                        <li v-for="(message, index) in toChat" v-bind:key="index" :class="message.owner">
                          <v-avatar :color="message.owner === 'me' ? '#eee': 'red'" size="36">
                            <span class="white--text">{{ message.owner }}</span>
                          </v-avatar>
                          <vue-typed-js :strings="[message.text]" :showCursor="false">
                                <p class="typing"></p>
                          </vue-typed-js>
                        </li>
                      </ul>
                </v-col>
              </v-row>
          </v-container>
          <v-footer fixed>
            <v-container class="ma-0 pa-0">
              <v-row no-gutters>
                <v-col>
                  <div class="d-flex flex-row align-center">
                     <v-alert type="info" v-if="isChatFinished" transition="scale-transition">
                        Chat finished. Thank you, it was nice to talk!
                     </v-alert>
                    <v-text-field v-model="input" placeholder="Type your message" @keypress.enter="send" @input="validateInput" v-if="!isChatFinished"></v-text-field>
                    <v-btn :disabled="isChatFinished || !isValid" @click="send" x-large class="chat-btn">{{buttonText}}</v-btn>
                  </div>
                </v-col>
              </v-row>
           </v-container>
          </v-footer>
    </v-content>
  </v-app>
</template>

<script>
import axios from 'axios'
export default {
  name: 'App',

  data: () => ({
    chatStarted: false,
    isValid: true,
    questionAsked: 0,
    infoMsg: '',
    userDetails: {},
    next: 0,
    input: '',
    toChat: [],
    messages: [],
  }),

  computed: {
    buttonText() {
      return this.chatStarted ? "Send Message" : "Let's chat"
    },
    isChatFinished() {
      return this.next >= this.messages.length;
    },
  },
  methods: {
    getBotMessages() {
        axios.get('./messages.json')
            .then(response => (this.messages = response.data));
    },
    validateInput() {
        if(!this.input) {
            this.isValid = false;
            this.infoMsg = 'Please enter your message first'
        }
        else {
            this.isValid = true;
            this.infoMsg = ''
        }
    },
    addMessageToChat(message) {
        this.toChat.push(message)
    },
    saveUserDetails(index) {
        let question = this.messages[index].ask;
        this.userDetails[question] = this.input
    },
    answer() {
        let active = true
        while(active) {
            if (typeof this.messages[this.next].ask === 'undefined') {
                this.addMessageToChat(this.messages[this.next]);
                this.questionAsked = false;
                this.next += 1;
            } else {
                this.addMessageToChat(this.messages[this.next]);
                this.questionAsked = this.next;
                this.next += 1;
                active = false;
            }
          }
    },
    send() {
      this.chatStarted = true;
      this.validateInput();
      if(this.isValid) {
          this.addMessageToChat({
            text: this.input,
            owner: 'me'
           });
          if(this.questionAsked) {
              this.saveUserDetails(this.questionAsked)
              this.input = ''
          }
          this.answer();
          this.input = '';
      }
      this.$nextTick(() => {
          this.$refs.chatbox.scrollTop = this.$refs.chatbox.scrollHeight + 90
        })
    },
  },
    mounted() {
      this.getBotMessages()
    }
};
</script>

<style>
  ul{
    list-style: none;
    margin: 0;
    padding: 0 30px 100px;
    position: absolute;
    left: 0;
    right: 0;
    overflow-y: scroll;
    height:600px;
    z-index: 0;
  }

  ul li{
    display:flex;
    align-items: center;
    clear: both;
    padding: 20px;
    border-radius: 30px;
    margin-bottom: 2px;
    font-family: Helvetica, Arial, sans-serif;
  }

  .him{
    background: #eee;
    float: left;
  }

  .me{
    float: right;
    background: #0084ff;
    color: #fff;
  }

  .him + .me{
    border-bottom-right-radius: 5px;
  }

  .me + .me{
    border-top-right-radius: 5px;
    border-bottom-right-radius: 5px;
  }

  .me:last-of-type {
    border-bottom-right-radius: 30px;
  }
  button.chat-btn {
    position: absolute;
    bottom: 15px;
    right: 25px;
  }
  .v-application p {
    margin-bottom: 0;
    }
  .typed-element p.typing {
      margin: 0 10px;
  }
</style>
