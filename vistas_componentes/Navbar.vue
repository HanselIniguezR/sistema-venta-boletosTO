<template>
  <nav v-if="showNavbar" :class="['navbar', 'invisible-navbar']">
    <router-link to="/" class="nav-link">Inicio</router-link>
    <router-link to="/signin" class="nav-link">SIGN IN</router-link>
    <router-link to="/signup" class="nav-link">SIGN UP</router-link>
    <button class="logout-btn" @click="handleLogout">Cerrar sesión</button>
  </nav>
</template>

<script setup>
import { computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useAuth } from '../composables/useAuth';

const { logout } = useAuth();
const route = useRoute();
const router = useRouter();

// Oculta la barra en '/' y '/home'
const showNavbar = computed(() => !(route.path === '/' || route.path === '/home'));

// Función que cierra sesión y redirige
const handleLogout = async () => {
  await logout();           // Ejecuta logout (borra storage y dispara evento)
  // Redirigir al Login principal
  try { router.replace('/'); } catch (e) { router.push('/'); }
};
</script>

<style scoped>
.navbar {
  display: flex;
  gap: 1rem;
  align-items: center;
  padding: 0.5rem 1rem;
  background: rgba(255, 255, 255, 0.95);
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  position: relative;
  z-index: 20;
}

.invisible-navbar {
  position: absolute;
  width: 0;
  height: 0;
  overflow: hidden;
  opacity: 0;
  pointer-events: none;
}

.nav-link {
  text-decoration: none;
  color: #333;
}

.logout-btn {
  margin-left: auto;
  padding: 0.4rem 0.8rem;
  cursor: pointer;
}
</style>