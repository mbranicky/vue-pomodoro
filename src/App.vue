<template>
  <center>
    <div id="app">
      <clock></clock>
    </div>
  </center>
</template>

<script>
import Clock from './components/Clock'

export default {
  name: 'app',
  components: {
    Clock
  }
}
</script>

<style>
html.md-theme-default {
  background-color: whitesmoke;
}
</style>

