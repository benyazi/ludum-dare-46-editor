<template>
  <div id="app">
    <MapBuilder msg="run"/>
  </div>
</template>

<script>
import MapBuilder from './components/MapBuilder.vue'

export default {
  name: 'app',
  components: {
    MapBuilder
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
