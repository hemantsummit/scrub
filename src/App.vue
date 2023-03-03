<template>
      <scrubDoc></scrubDoc>
</template>

<script>
import scrubDoc from "./components/Scrub-Doc.vue";
export default {
  components: {
      scrubDoc
  },
  data() {
      return {};
  },
  methods: {

  }
};
</script>
<style>
.wrapper {
  display: grid;
  grid-template-columns: 1fr 0.7fr;
}
</style>
