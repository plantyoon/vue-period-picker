<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <period-picker />
  </div>
</template>

<script>
import PeriodPicker from './components/PeriodPicker.vue';
export default {
  name: 'app',
  components: {
    PeriodPicker
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
