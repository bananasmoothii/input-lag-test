<script lang="ts">
import {defineComponent} from 'vue'

export default defineComponent({
  name: "NotFoundView"
})
</script>

<template>
  <div class="d-flex align-items-center justify-content-center">
    <div class="text-center">
      <h1 class="display-1 fw-bold">404</h1>
      <p class="fs-3">Oops! Page not found.</p>
      <p class="lead">
        The page <code>{{ this.$route.path }}</code> doesn't exist.
      </p>
      <RouterLink class="btn btn-primary" to="/">Go Home</RouterLink>
    </div>
  </div>
</template>

<style lang="scss" scoped>

</style>
