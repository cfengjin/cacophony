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
  background-color: whitesmoke;

  display: flex;
  flex-direction: column;
  justify-content: flex-end;

  height: 100vh;
}

body {
  margin: 0;
}
</style>
