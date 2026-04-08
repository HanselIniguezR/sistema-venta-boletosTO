<template>
  <div class="home-page">
    <div class="monogram-bg"></div>
    <aside class="sidebar" @mouseover="expandMenu" @mouseleave="collapseMenu" :class="{ expanded: isExpanded }">
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
      <button @click="logout" class="btn-logout" v-if="isExpanded">Cerrar sesión</button>
    </aside>
    <main class="main-content">
      <div class="compras-content">
        <h1>Mis Compras</h1>
        <table class="compras-table">
          <thead>
            <tr>
              <th>Evento</th>
              <th>Confirmación</th>
              <th>Fecha</th>
              <th>Hora</th>
              <th>Asiento</th>
              <th>Precio</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="loading">
              <td colspan="5">Cargando boletos...</td>
            </tr>
            <tr v-else-if="error">
              <td colspan="5">Error: {{ error }}</td>
            </tr>
            <tr v-else-if="tickets.length === 0">
              <td colspan="5">No tienes boletos aún.</td>
            </tr>
            <tr v-else v-for="t in tickets" :key="t.id">
              <td>{{ t.evento_title }}</td>
              <td class="order-code">{{ t.order_code || ('#' + t.id) }}</td>
              <td>{{ t.fecha || '-' }}</td>
              <td>{{ t.hora || '-' }}</td>
              <td>{{ t.asientos || '-' }}</td>
              <td>
                ${{ Number(t.total || 0).toFixed(2) }}
                <span v-if="t.pdf_path"> &nbsp; <a :href="API_BASE + t.pdf_path" target="_blank" rel="noopener">Ver PDF</a></span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const API_BASE = import.meta.env.VITE_API_BASE || 'http://localhost:3001';

const isExpanded = ref(false);

function expandMenu() {
  isExpanded.value = true;
}
function collapseMenu() {
  isExpanded.value = false;
}
function logout() {
  localStorage.removeItem('isAuthenticated');
  window.location.href = '/signin';
}

// Tickets state
const tickets = ref([]);
const loading = ref(false);
const error = ref('');

async function loadTickets() {
  try {
    loading.value = true;
    error.value = '';
    const rawUser = localStorage.getItem('user');
    if (!rawUser) {
      tickets.value = [];
      loading.value = false;
      return;
    }
    const user = JSON.parse(rawUser);
    if (!user || !user.id) {
      tickets.value = [];
      loading.value = false;
      return;
    }
  // Prefer querying by user_id; if not available, fall back to email so older orders without user_id can be shown
  const queryParam = user.id ? `user_id=${encodeURIComponent(user.id)}` : `email=${encodeURIComponent(user.correo_electronico || user.email || '')}`;
  const res = await fetch(`${API_BASE}/api/mis-boletos?${queryParam}`);
    if (!res.ok) throw new Error('Error al obtener boletos');
    const json = await res.json();
    tickets.value = json.tickets || [];
  } catch (e) {
    console.error('loadTickets error', e);
    error.value = String(e.message || e);
  } finally {
    loading.value = false;
  }
}

const images = [
  '/img1.jpg',
  '/img2.jpg',
  '/img3.jpg'
];
const currentIndex = ref(0);
const currentImage = ref(images[0]);
let intervalId = null;

onMounted(() => {
  intervalId = setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % images.length;
    currentImage.value = images[currentIndex.value];
  }, 20000); // 20 segundos
  loadTickets();
});
onBeforeUnmount(() => {
  clearInterval(intervalId);
});
</script>

<style scoped>
.home-page {
  display: flex;
  height: 100vh;
  background: linear-gradient(120deg, #FFFDD0 60%, #e6e6e6 100%);
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
.sidebar {
  background: #556B2F;
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
  color: #FFD700;
}
.btn-logout {
  background: #fff;
  color: #556B2F;
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
.main-content {
  flex: 1;
  padding: 3rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  z-index: 1;
  position: relative;
}
.compras-content {
  z-index: 2;
  margin-top: 80px;
  margin-bottom: 60px;
  width: 100%;
  max-width: 900px;
  background: rgba(255,255,255,0.95);
  border-radius: 24px;
  box-shadow: 0 4px 18px rgba(85, 107, 47, 0.10);
  padding: 2.5rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-left: auto;
  margin-right: auto;
}
.compras-content h1 {
  font-size: 2.1rem;
  color: #556B2F;
  margin-bottom: 1.5rem;
  font-weight: 600;
  text-align: center;
}
.compras-table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px #bdb76b22;
}
.compras-table th,
.compras-table td {
  padding: 1rem;
  border-bottom: 1px solid #e0e0b0;
  text-align: left;
  font-size: 1rem;
}
.compras-table th {
  background: #f8f8e7;
  color: #556B2F;
  font-weight: 600;
}
.compras-table tr:last-child td {
  border-bottom: none;
}
</style>