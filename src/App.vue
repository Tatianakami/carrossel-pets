<template>
  <div id="app">
    <h1>🐾 Carrossel de Pets</h1>
    <Carrossel />
  </div>
</template>

<script>
import Carrossel from "./components/Carrossel.vue";

export default {
  name: "App",
  components: {
    Carrossel,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  padding: 20px;
  background: linear-gradient(135deg, #f0f0f0, #cceaff);
  min-height: 100vh;
}
</style>
