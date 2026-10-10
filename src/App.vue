<script setup>
import { RouterLink, RouterView } from "vue-router";
// import HelloWorld from "./components/HelloWorld.vue";
</script>

<template>
  <header>
    <div id="app">
      <nav class="navbar">
        <router-link to="/" class="nav-link">Tax Calculator</router-link>
        <!-- <router-link to="/home" class="nav-link">Home</router-link>
        <router-link to="/about" class="nav-link">About</router-link> -->
      </nav>
      <RouterView />
    </div>
  </header>
</template>

<style scoped>
#app {
  font-family: "Arial", sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.navbar {
  background-color: #333;
  width: 100%;
  padding: 10px;
  display: flex;
  justify-content: center;
}

.nav-link {
  color: white;
  padding: 10px 20px;
  text-decoration: none;
  margin: 0 10px;
  border-radius: 5px;
  transition: background-color 0.3s;
}

.nav-link:hover {
  background-color: #555;
}

router-view {
  margin-top: 20px;
}
</style>
