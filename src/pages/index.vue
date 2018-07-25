<template>
  <section class="section about">
    <div class="container is-fluid">
      <h1 class="about__title is-size-2 has-text-weight-bold">About</h1>
      <h2 class="subtitle">
        A simple container to divide your page into <strong>sections</strong>, like the one you're currently reading
      </h2>
    </div>
  </section>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
export default {
  name: 'home',
  components: {
    HelloWorld
  }
}
</script>

<style scoped>
.about__title {
  margin-bottom: 46px;
}
</style>
