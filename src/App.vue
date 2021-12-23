<template>
  <div id="app">
    <CheatTable />
  </div>
</template>

<script>
import CheatTable from "./components/CheatTable.vue";

export default {
  name: "App",
  components: {
    CheatTable,
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
