
<template>
  <div class="AltChat">
    <div id="chatbox_1" ref="box" class="chat-box">
      <p id="prg"
          v-for="(message, i) in messages"
          :key="i"
          :class="{
          message: true,
          me: message.participantId === $magpie.socket.participantId
        }"
          v-text="message.message"
      ></p>
    </div>
    <div class="notice-box">{{active}}</div>
    <div class="chat-input">
      <textarea
          ref="text"
          cols="50"
          placeholder="Type your message to the other participant here."
          @keydown.enter="send"
      ></textarea>
      <button @click.stop="send()" @v>
        Send
      </button>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
const EVENT_CHAT_MESSAGE = 'chat_message';
const EVENT_CHAT_ACTIVE = 'inactive';

export default {
  name: 'AltChat',
  data() {
    return {
      messages: [],
      statusMonitor: {},
      active: "Your chat partner has not yet entered the chat."
    };
  },
  socket: {
    [EVENT_CHAT_MESSAGE](payload) {
      this.messages.push(payload);
      Vue.nextTick(() => {
        this.$refs.box.scrollTop = this.$refs.box.scrollHeight;
      });
      this.$emit('update:messages', this.messages);
    },
    [EVENT_CHAT_ACTIVE](payload) {
      this.statusMonitor[payload.participantId] = payload;
      this.$emit('update:statusMonitor', this.statusMonitor);
    }
  },
  EVENT_CHAT_MESSAGE,
  EVENT_CHAT_ACTIVE,
  created () {
    this.monitorActivity()
  },
  deactivated() {
    clearInterval(this.update);
  },
  methods: {
    send() {
      const message = this.$refs.text.value;
      if (!message) {
        return;
      }
      this.$magpie.socket.broadcast(EVENT_CHAT_MESSAGE, {
        message,
        participantId: this.$magpie.socket.participantId
      });
      this.$refs.text.value = '';
      this.$refs.text.focus();
    },
    monitorActivity(){
      this.update = setInterval(() => {
        // set yourself active
        this.$magpie.socket.broadcast(EVENT_CHAT_ACTIVE, {
          status: "active",
          participantId: this.$magpie.socket.participantId,
          lastUpdated: new Date()
        });
        // check if partner is active
        for (let [participantID, status] of Object.entries(this.statusMonitor)) {
          console.log(this.statusMonitor)
          if (participantID != this.$magpie.socket.participantId) {
            if (status.status === "active") {
              if ((new Date() - new Date(status.lastUpdated)) > 15 * 1000) {
                // If there's no response after 30 seconds, partner has left the chat
                this.active = "Your chat partner has left the chat. Please click [leave chat] to finish the experiment.";
              } else {
                this.active = "Your chat partner is active";
              }
            }
          }
          console.log(this.active);
        }
      }, 3000)
    }
  }
};
</script>


<style>
.chat {
  width: 450px;
}

.chat-box {
  overflow: auto;
  height: 400px;
}

.message {
  width: 55%;
  float: left;
  text-align: left;
  background: #5187ba7a;
  border-radius: 4px;
  padding: 4px;
  margin: 5px 0;
}

.message.me {
  float: right;
  background: #70ba517a;
}

.chat-input {
  display: flex;
}

.chat-input textarea {
  flex-grow: 1;
  height: 32px;
}

.chat-input button {
  margin: 0;
}

.notice-box:hover { background-color: #ff9c00; }
.notice-box {
  background-color: #e8a235;
  border-radius: 4px;
  color: #fefefb;
  font-weight:bold;
  text-transform:uppercase;
  letter-spacing: 1px;
}

</style>
