<template>
  <div>
    <ClientOnly>
      <v-app id="app">
        <!-- app -->
        <NuxtPage /> </v-app
    ></ClientOnly>
  </div>
</template>
<style>
body {
  margin: 0;
  padding: 0;
}
#app {
  height: 100%;
}
</style>
