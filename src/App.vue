<script lang="ts">

</script>

<template>
  <nav class="navbar bg-light">
    <div class="container-fluid">
      <RouterLink class="navbar-brand" to="/">
        <img alt="Bootstrap" height="24" src="@/assets/logo.svg" width="30">
        Input Lag Test
      </RouterLink>
    </div>
  </nav>
  <div class="router-view  mx-auto px-5 mt-4">
    <RouterView/>
  </div>
</template>

<style lang="scss">
@import "scss/custom";
</style>
