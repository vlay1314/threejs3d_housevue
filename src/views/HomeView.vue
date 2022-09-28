<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

// eslint-disable-next-line vue/no-export-in-script-setup
export default {
  name: 'HomeView',
  components: {
    HelloWorld
  }
}
</script>
