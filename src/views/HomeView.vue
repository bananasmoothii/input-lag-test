<template>
  <p>
    Test your input lag from the moment you click/tap/press a button to the moment you see the result.
  </p>
  <p>
    <strong>How to use:</strong> Press any button when the cursor gets on a black square and the area turns darker
    release when the cursor is not on a black square. The yellow squares are when you actually pressed the button.
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
