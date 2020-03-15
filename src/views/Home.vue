<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png" />
    <CountryData country="denmark" />
  </div>
</template>

<script>
// @ is an alias to /src
import CountryData from "@/components/CountryData.vue";

export default {
  name: "Home",
  components: {
    CountryData
  }
};
</script>
