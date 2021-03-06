<template>
  <div id="app">
    <Grid />
  </div>
</template>

<script>
import Grid from './components/Grid.vue'

export default {
  name: 'App',
  components: {
    Grid
  }
}
</script>

<style>

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
  background-color: #f4eeff
}
</style>
