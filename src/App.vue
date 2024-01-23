<template>
  <div id="app">
    <OfficeMap />
  </div>
</template>

<script>
import OfficeMap from '@/components/OfficeMap.vue';

export default {
  name: 'App',
  components: {
    OfficeMap,
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
