<template>
  <div class="max-w-5xl mx-auto px-6 sm:px-16 py-16">
    <CountryData />
    <Recommendations />
  </div>
</template>

<script>
// @ is an alias to /src
import CountryData from "@/components/CountryData.vue";
import Recommendations from "@/components/Recommendations.vue";

export default {
  name: "Home",
  components: {
    CountryData,
    Recommendations
  }
};
</script>
