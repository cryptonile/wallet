<template>
  <div id="app" data-app>
    <Main/>
  </div>
</template>

<script>
import Main from './components/Main.vue'

export default {
  name: 'app',
  components: {
    Main
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
.swal-text {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  font-size: 18px;
}
.swal-button {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  background-color: rgba(0,0,0,.12);
  color: black;
}
.swal-content__input:focus, .swal-content__textarea:focus {
  border-color: black;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
}
</style>