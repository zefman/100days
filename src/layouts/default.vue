<template>
  <div>
    <nav class="navbar is-transparent" role="navigation" aria-label="main navigation">
      <div class="navbar-brand">
        <a class="navbar-item" href="/">
          <h1 class="is-size-1 has-text-weight-bold has-text-white">#100DaysOfCode</h1>
        </a>
      </div>

      <div class="navbar-end">
        <p class="navbar-item"><a href="https://twitter.com/jozefmaxted">Twitter: @jozefmaxted</a></p>
        <p class="navbar-item"><a>Github</a></p>
      </div>
    </nav>

    <router-view/>
  </div>
</template>

<style scoped>
.navbar {
  background-color: rgba(0,0,0, 0.1);
}
</style>
