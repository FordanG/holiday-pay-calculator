<template>
  <div class="container">
    <header>
      <Menubar>
        <template #start>
          <h1>2024 Philippines Holiday Pay Calculator</h1>
          <!-- <img src="logo.png" alt="Logo" height="40" class="mr-2" /> -->
        </template>
      </Menubar>
    </header>

    <main>
      <div class="content">
        <slot />
      </div>
    </main>

    <footer>
      <p>&copy; 2024 Holiday Pay Calculator. All rights reserved.</p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from "vue";
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

header {
  margin-bottom: 20px;
}

main {
  margin-top: 20px;
}

.content {
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

footer {
  margin-top: 20px;
  text-align: center;
  color: #777;
  font-size: 14px;
}
</style>
