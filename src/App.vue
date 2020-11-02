<template>
  <div id="app">
    <amplify-authenticator>
      <div>
        <h1>Hey, {{user.username}}!</h1>
        <hello-world />
        <amplify-sign-out></amplify-sign-out>
      </div>
    </amplify-authenticator>
  </div>
</template>

<script>
import HelloWorld from './HelloWorld.vue'
import { AuthState, onAuthUIStateChange } from '@aws-amplify/ui-components'

export default {
  name: 'app',
  data() {
    return {
      user: { },
    }
  },
  components: {
    HelloWorld
  },
  created() {
    // authentication state managament
    onAuthUIStateChange((state, user) => {
      // set current user and load data after login
      if (state === AuthState.SignedIn) {
        this.user = user;
      }
    })
  }
}
</script>
