<template>
  <div class="navbar">
    <h1 class="navbar-header--text">
      Notes App
    </h1>
    <nav>
      <NLink class="nav-link" to="/">
        home
      </NLink>
      <NLink class="nav-link" to="/about">
        about
      </NLink>
      <NLink class="nav-link" to="/documents">
        documents
      </NLink>
    </nav>
  </div>
</template>

<style scoped lang='postcss'>
.navbar {
  grid-area: "nav";
  @apply min-h-screen w-full bg-purple-dark text-white px-8;
}
.navbar-header--text {
  @apply my-8;
}
nav {
  @apply flex flex-col mt-20;
}
.nav-link {
  @apply w-full capitalize;
}
</style>
