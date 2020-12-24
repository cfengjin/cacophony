<template>
  <Keyboard />
</template>

<script>
import Keyboard from './components/Keyboard.vue'

export default {
  name: 'App',
  components: {
    Keyboard
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
}
</style>
