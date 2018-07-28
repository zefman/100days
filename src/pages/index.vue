<template>
  <section class="section about">
    <div class="container is-fluid">
      <div class="columns is-6 is-variable">
        <div class="column">
          <h1 class="about__title is-size-2 has-text-weight-bold">About</h1>
          <p class="is-size-4">
            I am attempting to finally learn WebGL after half heartedly trying many a time. I'll be useing the <a href="http://www.100daysofcode.com/" target="_blank">#100DaysOfCode</a>
            project as way to finally help me crack WebGL. This site will serve as an archive of my learnings and experirments
            along the way.
          </p>
        </div>
        <div class="column">
          <h2 class="about__title is-size-2 has-text-weight-bold">Days:</h2>

          <ul>
            <li class="subtitle" v-for="(day, index) in days" :key="day.name"><router-link :to="day.path">Day {{ index + 1 }}</router-link></li>
          </ul>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: 'home',
  mounted () {
    console.log(this.$router.options.routes)
  },
  computed: {
    days () {
      return this.$router.options.routes[0].children.filter(child => child.name.includes('days'))
    }
  }
}
</script>

<style scoped>
.about__title {
  margin-bottom: 46px;
}
</style>
