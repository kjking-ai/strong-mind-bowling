<template>
  <div id="app">
    <Bowling></Bowling>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Bowling from './components/Bowling.vue';

@Component({
  components: {
    Bowling,
  },
})
export default class App extends Vue {}
</script>

<style>
  body {
    padding: 40px;
  }
</style>
