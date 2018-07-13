<template>
  <div id="app">
    <AlarmRuleAdd />
  </div>
</template>

<script>
import AlarmRuleAdd from './components/alarmRuleAdd'

export default {
  name: 'app',
  components: {
    AlarmRuleAdd
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
