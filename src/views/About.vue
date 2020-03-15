<template>
  <div class="about">
    <h1>This is a cool page</h1>
    <test-machine />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import TestMachine from "@/components/TestMachine.vue";
export default Vue.extend({
  components: {
    TestMachine
  }
});
</script>
