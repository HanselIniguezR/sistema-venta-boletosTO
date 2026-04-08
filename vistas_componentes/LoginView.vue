<template>
  <div class="login-page" :class="{ 'lift-animation': isAnimating }">
    <!-- Si la barra de navegación global está visible o el usuario ya está autenticado,
         no mostramos este nav duplicado -->
    <nav v-if="!globalNavbarVisible && !isAuthenticated" class="nav-links">
      <a href="#" class="nav-item" @click.prevent="startAnimation('/signin')">SIGN IN</a>
      <a href="#" class="nav-item" @click.prevent="startAnimation('/signup')">SIGN UP</a>
      <a href="#" class="nav-item">SERVICE</a>
    </nav>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useRouter, useRoute } from 'vue-router';

const isAnimating = ref(false);
const router = useRouter();
const route = useRoute();

// si la app usa el mismo criterio que el Navbar para ocultarlo
const globalNavbarVisible = computed(() => !(route.path === '/' || route.path === '/home'));

// estado de autenticación (mismo criterio que Navbar)
const isAuthenticated = computed(() => !!localStorage.getItem('token') || !!localStorage.getItem('user'));

function startAnimation(path) {
  isAnimating.value = true;
  setTimeout(() => {
    router.push(path);
  }, 1200);
}

// Exponer startAnimation para que sea accesible desde tests/depuración
defineExpose({ startAnimation });
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;500&display=swap');

.login-page {
  height: 100vh;
  background-image: url('/logodos.png');
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  overflow: hidden;
  position: relative;
  padding: 2rem;
  transition: transform 1.2s ease, opacity 1.2s ease;
  transform-origin: center top;
  perspective: 1000px;
}

.lift-animation {
  transform: translateY(-100%) rotateX(-75deg);
  opacity: 0;
}

.nav-links {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
}

.nav-item {
  color: #FFFDD0;
  text-decoration: none;
  font-weight: 500;
  letter-spacing: 0.5px;
  font-size: 1rem;
  font-family: 'Poppins', sans-serif;
  position: relative;
  transition: transform 0.3s ease, color 0.3s ease;
}

.nav-item:hover {
  transform: translateY(-5px);
  color: #FFD700;
}
</style>