<template>
  <div class="main">
    <ColorPicker msg="Welcome to Your Vue.js App" canva_len="300" />
  </div>
</template>

<script>
import ColorPicker from "./components/colorPicker.vue";

export default {
  name: "App",
  components: {
    ColorPicker,
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
}
.main {
  display: flex;
  flex-direction: row;
  justify-content: center;
}
</style>
