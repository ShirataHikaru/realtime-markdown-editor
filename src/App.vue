<template>
  <div id="app">
    <md-editor />
  </div>
</template>

<script>
import MdEditor from './components/MdEditor.vue'

export default {
  name: 'app',
  components: {
    MdEditor
  }
}
</script>

<style>
html, body, #app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100%;
}

#app{
  margin-top: 60px;
}
</style>
