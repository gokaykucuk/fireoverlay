<template>
  <div id="app">
    <div id="wrapper">
      <div id="data"></div>
      <div id="map"><Map></Map></div>
    </div>
  </div>
</template>

<script>
import Map from "./components/Map.vue";

export default {
  name: "app",
  components: {
    Map
  },
  mounted() {
    //console.log(cities);
  }
};
</script>

<style>
html,
body,
#app,
#wrapper {
  height: 100%;
  margin: 0;
}
#wrapper {
  display: grid;
  grid-template-columns: repeat(8, [col-start] 1fr);
}
#data {
  grid-column: col-start 1 / span 3;
  grid-row: 1;
}
#map {
  grid-column: col-start 3 / span 7;
  grid-row: 1;
}
</style>