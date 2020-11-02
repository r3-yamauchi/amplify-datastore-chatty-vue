<template>
  <div>
    <form v-on:submit.prevent>
      <input v-model="form.message" />
      <button @click="sendMessage">Send</button>
    </form>

    <div v-for="message of sorted" :key="message.id">
      <div>{{ message.message }} {{ message.user }} - {{ message.createdAt }}</div>
    </div>

    <button @click="allDel">Delete all messages.</button>

  </div>
</template>
<script>
import { Auth } from "aws-amplify";
import { DataStore, Predicates } from "@aws-amplify/datastore";
import { Chatty } from "./models";

export default {
  name: 'app',
  data() {
    return {
      user: {},
      messages: [],
      form: {},
      subscription: undefined
    }
  },
  computed: {
    sorted() {
      return [...this.messages].sort((a, b) => -a.createdAt.localeCompare(b.createdAt));
    }
  },
  created() {
    this.currentUser();
    //Subscribe to changes
    this.subscription = DataStore.observe(Chatty).subscribe(msg => {
      console.log(msg.model, msg.opType, msg.element);
      this.loadMessages();
    });
  },
  destroyed() {
    if (!this.subscription) return;
    this.subscription.unsubscribe();
  },
  methods: {
    sendMessage() {
      const { message } = this.form
      if (!message) return;
      DataStore.save(new Chatty({
        user: this.user.username,
        message: message,
        createdAt: new Date().toISOString()
      })).then(() => {
        this.form = { message: '' };
        this.loadMessages();
      }).catch(e => {
        console.log('error creating message...', e);
      });
    },
    currentUser() {
      Auth.currentAuthenticatedUser().then(user => {
        this.user = user;
        this.loadMessages();
      });
    },
    loadMessages() {
      DataStore.query(Chatty, Predicates.ALL).then(messages => {
        this.messages = messages;
      });
    },
    allDel() {
      DataStore.delete(Chatty, Predicates.ALL).then(() => {
        console.log('messages deleted!');
        this.loadMessages();
      });
    }
  }
}
</script>
