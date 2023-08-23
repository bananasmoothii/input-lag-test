<script lang="ts">
import {defineComponent} from 'vue';

export default defineComponent({
  name: 'App',

  mounted() {
    const noscript = document.querySelectorAll('noscript');
    noscript.forEach((element) => {
      element.remove();
    });
  }
});
</script>

<template>
  <nav class="navbar bg-light">
    <div class="container-fluid">
      <RouterLink class="navbar-brand" to="/">
        <img alt="Logo" class="mx-3" height="27" src="@/assets/logo.svg" width="27">
        <span>Input Lag Test</span>
      </RouterLink>
      <div class="text-secondary ms-auto">
        Made by <a class="git-profile-link text-secondary" href="https://github.com/bananasmoothii">Bananasmoothii</a>
        <a href="https://github.com/bananasmoothii/input-lag-test">
          <img alt="GitHub" class="mx-3" height="27.2" src="@/assets/github-mark.svg" width="27.2">
        </a>
      </div>
    </div>
  </nav>
  <div class="router-view mx-auto px-5 mt-4">
    <RouterView/>
  </div>
</template>

<style lang="scss">
@import "scss/custom";

.git-profile-link {
  text-decoration: underline;
  transition: color 0.2s ease-in-out;

  &:hover {
    color: #000 !important;
  }
}
</style>
