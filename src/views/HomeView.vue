<template>
  <p>
    Test your input lag from the moment you click/tap/press a button to the moment you see the result.
  </p>
  <p>
    This test is not very precise, as it takes <i>you</i> into account. Try to be as consistent as possible.
  </p>
  <p>
    <strong>How to use:</strong> Press any button when a black rectangle is on the middle line (and the area turns
    darker), release the button or your finger when the rectangle isn't on the middle line anymore. The yellow
    rectangles show when you actually pressed or tapped the button.
  </p>

  <InputLagGame/>
</template>

<script lang="ts">
import {defineComponent} from 'vue';
import InputLagGame from "@/components/InputLagGame.vue";

export default defineComponent({
  name: 'HomeView',
  components: {InputLagGame},
});
</script>
