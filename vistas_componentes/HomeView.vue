<template>
  <div class="home-page">
    <div class="monogram-bg" aria-hidden="true"></div>

    <aside
      class="sidebar"
      @mouseover="expandMenu"
      @mouseleave="collapseMenu"
      :class="{ expanded: isExpanded }"
    >
      <h2 v-if="isExpanded">Categorías</h2>
      <ul>
        <li>
          <router-link to="/home" class="menu-item" active-class="active">
            <span v-if="isExpanded">Página Principal</span>
          </router-link>
        </li>
        <li>
          <router-link to="/teatro" class="menu-item" active-class="active">
            <span v-if="isExpanded">Teatro</span>
          </router-link>
        </li>
        <li>
          <router-link to="/cine" class="menu-item" active-class="active">
            <span v-if="isExpanded">Cine</span>
          </router-link>
        </li>
        <li>
          <router-link to="/museo" class="menu-item" active-class="active">
            <span v-if="isExpanded">Museos</span>
          </router-link>
        </li>
        <li>
          <router-link to="/mis-boletos" class="menu-item" active-class="active">
            <span v-if="isExpanded">Mis boletos</span>
          </router-link>
        </li>
        <li>
          <router-link to="/mi-perfil" class="menu-item" active-class="active">
            <span v-if="isExpanded">Mi perfil</span>
          </router-link>
        </li>
      </ul>

      <button @click="handleLogout" class="btn-logout" v-if="isExpanded">Cerrar sesión</button>
    </aside>

    <main class="main-content">
      <div class="image-carousel" role="img" aria-label="Carrusel de imágenes">
        <img :src="currentImage" alt="Imagen destacada" />
      </div>

      <div class="welcome-container">
        <h1>Bienvenido a Ticket Off</h1>
        <p>Tu mejor opción para comprar boletos</p>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const isExpanded = ref(false);

function expandMenu() {
  isExpanded.value = true;
}
function collapseMenu() {
  isExpanded.value = false;
}

function handleLogout() {
  try {
    localStorage.removeItem('isAuthenticated');
    localStorage.removeItem('token');
    localStorage.removeItem('user');
    sessionStorage.clear();
  } catch {
    // ignorar errores
  }
  router.push('/'); // redirige a LoginView ("/")
}

// Carrusel de imágenes
const images = ['/img1.jpg', '/img2.jpg', '/img3.jpg'];
const currentIndex = ref(0);
const currentImage = ref(images[0]);
let intervalId = null;

onMounted(() => {
  intervalId = setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % images.length;
    currentImage.value = images[currentIndex.value];
  }, 20000); // cada 20 segundos
});

onBeforeUnmount(() => {
  if (intervalId) clearInterval(intervalId);
});
</script>

<style scoped>
.home-page {
  display: flex;
  height: 100vh;
  background: linear-gradient(120deg, #fffdd0 60%, #e6e6e6 100%);
  font-family: 'Poppins', sans-serif;
  position: relative;
  overflow: hidden;
}

.monogram-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  opacity: 0.07;
  background-image: url('/monogram.png');
  background-repeat: repeat;
  background-size: 120px 120px;
  background-position: center;
  filter: grayscale(1) brightness(1.2);
}

/* Sidebar */
.sidebar {
  background: #556b2f;
  color: #fff;
  width: 70px;
  min-width: 70px;
  transition: width 0.3s;
  padding-top: 40px;
  padding-bottom: 40px;
  box-shadow: 2px 0 8px rgba(85, 107, 47, 0.08);
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 2;
}
.sidebar.expanded {
  width: 220px;
  min-width: 220px;
  align-items: flex-start;
}
.sidebar h2 {
  margin-bottom: 2rem;
  font-size: 1.1rem;
  font-weight: 500;
  letter-spacing: 1px;
  color: #fff;
  text-align: left;
  width: 100%;
  padding-left: 2rem;
  display: none;
}
.sidebar.expanded h2 {
  display: block;
}
.sidebar ul {
  list-style: none;
  padding: 0;
  margin: 0;
  width: 100%;
}
.menu-item {
  display: flex;
  align-items: center;
  width: 100%;
  padding: 1.1rem 0.5rem;
  color: #fff;
  text-decoration: none;
  font-size: 0;
  border: none;
  background: none;
  border-radius: 0;
  border-bottom: 1px solid #6b8e23;
  transition: background 0.2s, color 0.2s, font-size 0.3s, padding 0.3s;
  cursor: pointer;
  font-weight: 500;
  letter-spacing: 0.5px;
  text-align: left;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.menu-item:last-child {
  border-bottom: none;
}
.sidebar.expanded .menu-item {
  font-size: 1.08rem;
  padding: 1.1rem 2rem;
}
.menu-item:hover,
.menu-item.active,
.router-link-active.menu-item {
  background: #445522;
  color: #ffd700;
}
.btn-logout {
  background: #fff;
  color: #556b2f;
  border: none;
  border-radius: 6px;
  padding: 0.7rem 1.5rem;
  font-size: 1rem;
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
  margin-top: 2rem;
  width: 80%;
  align-self: center;
  display: none;
}
.sidebar.expanded .btn-logout {
  display: block;
}

/* Main content */
.main-content {
  flex: 1;
  padding: 3rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  z-index: 1;
  position: relative;
}
.image-carousel {
  width: 900px;
  height: 500px;
  margin-bottom: 2rem;
  border-radius: 28px;
  overflow: hidden;
  box-shadow: 0 4px 18px rgba(85, 107, 47, 0.12);
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  max-width: 95vw;
  max-height: 60vh;
}
.image-carousel img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: opacity 0.5s;
}
.welcome-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  margin-bottom: 40px;
}
.welcome-container h1 {
  font-size: 2.2rem;
  font-weight: bold;
  color: #333;
  margin-bottom: 0.7rem;
  text-align: center;
}
.welcome-container p {
  font-size: 1.1rem;
  color: #444;
  margin-bottom: 0;
  text-align: center;
}

/* Responsive */
@media (max-width: 900px) {
  .sidebar { display: none; }
  .image-carousel { width: 100%; height: 320px; }
  .main-content { padding: 1.25rem; }
}
</style>
