<template>
  <div id="app">
    <Scene />
  </div>
</template>

<script>
import Scene from './components/Scene.vue'

export default {
  name: 'App',
  components: {
    Scene
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
  margin: 0px;
  padding: 0px;
  height: 100vh;
  background-color: #2c3e50;
}
</style>
