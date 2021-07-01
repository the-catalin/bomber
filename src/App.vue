<template>
  <div id="app">
    <FlipBoard/>
  </div>
</template>

<script>
import FlipBoard from './components/FlipBoard.vue'

export default {
  name: 'app',
  components: {
    FlipBoard
  }
}
</script>

<style>
body {
  margin: 0;
}
#app {
  font-family: 'Roboto', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*color: #2c3e50;*/
}
</style>
