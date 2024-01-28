<template>
  <div id="app"  @mousemove="updateCursorPosition">
    <h1>Welcome to the Office Map</h1>
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
  data() {
    return {
      cursorX: 0,
      cursorY: 0,
    };
  },
  methods: {
    
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  max-width: 100vw;
  width: 100vw;
  display: block;
  padding: 0;
  margin: 0;
}

h1{
  margin: 10 !important;
  padding: 0 !important;
}

</style>
