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
      <div class="perfil-content">
        <h1>Mi Perfil</h1>
        <div class="perfil-table-wrapper">
          <table class="perfil-table">
            <thead>
              <tr>
                <th>Campo</th>
                <th>Valor</th>
                <th>Acción</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>Nombre</td>
                <td>
                  <input 
                    v-if="campoEditando === 'nombre'" 
                    v-model="perfil.nombre" 
                    @keyup.enter="guardar('nombre')"
                    type="text"
                    class="edit-input"
                  />
                  <span v-else>{{ perfil.nombre }}</span>
                </td>
                <td>
                  <button class="edit-btn" @click="alternarEdicion('nombre')" title="Editar nombre">
                    <span v-if="campoEditando === 'nombre'">💾</span>
                    <span v-else>✏️</span>
                  </button>
                </td>
              </tr>
              
              <tr>
                <td>Correo electrónico</td>
                <td>
                  <input 
                    v-if="campoEditando === 'email'" 
                    v-model="perfil.email" 
                    @keyup.enter="guardar('email')"
                    type="email"
                    class="edit-input"
                  />
                  <span v-else>{{ perfil.email }}</span>
                </td>
                <td>
                  <button class="edit-btn" @click="alternarEdicion('email')" title="Editar correo">
                    <span v-if="campoEditando === 'email'">💾</span>
                    <span v-else>✏️</span>
                  </button>
                </td>
              </tr>
              
              <tr>
                <td>Contraseña</td>
                <td>********</td>
                <td>
                  <button class="edit-btn" @click="editarContrasena" title="Cambiar contraseña">
                    ✏️
                  </button>
                </td>
              </tr>
              
              <tr>
                <td>Tarjetas guardadas</td>
                <td>
                  <ul class="card-list">
                    <li v-for="(tarjeta, idx) in perfil.tarjetas" :key="idx">
                      {{ tarjeta }}
                    </li>
                  </ul>
                  <div v-if="campoEditando === 'tarjetas'" class="edit-hint">
                    Presiona 💾 para guardar la lista de tarjetas editada.
                  </div>
                </td>
                <td>
                  <button class="edit-btn" @click="alternarEdicion('tarjetas')" title="Editar tarjetas">
                    <span v-if="campoEditando === 'tarjetas'">💾</span>
                    <span v-else>✏️</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
          
          <button v-if="campoEditando" @click="campoEditando = null" class="btn-cancelar">
            Cancelar Edición
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const isExpanded = ref(false);
const campoEditando = ref(null);
const perfil = ref({
  nombre: 'Juan Pérez',
  email: 'juan.perez@email.com',
  passwordActual: 'mipass123', 
  tarjetas: ['**** **** **** 1234', '**** **** **** 5678']
});

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

function alternarEdicion(campo) {
  if (campoEditando.value === campo) {
    guardar(campo);
  } else {
    if (campo === 'tarjetas') {
      if (editarTarjetas()) {
        campoEditando.value = campo;
      }
    } else {
      campoEditando.value = campo;
      setTimeout(() => {
        const input = document.querySelector('.edit-input');
        if (input) input.focus();
      }, 0);
    }
  }
}

function guardar(campo) {
  console.log(`Datos de ${campo} guardados:`, perfil.value[campo]);
  campoEditando.value = null; 
}

function editarContrasena() {
  const nuevaPass = prompt("Ingresa tu nueva contraseña:");
  if (nuevaPass) {
    perfil.value.passwordActual = nuevaPass; 
    alert("Contraseña actualizada con éxito.");
  }
}

function editarTarjetas() {
  const tarjetasString = perfil.value.tarjetas.join('\n');
  const nuevoString = prompt("Edita tus tarjetas (una por línea):", tarjetasString);
  
  if (nuevoString !== null) {
    perfil.value.tarjetas = nuevoString.split('\n')
      .map(s => s.trim())
      .filter(s => s.length > 0);
    return true;
  } else {
    campoEditando.value = null;
    return false;
  }
}
</script>

<style scoped>
/* ESTILOS (Sintaxis revisada y corregida para eliminar el carácter invisible) */
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
.perfil-content {
  z-index: 2;
  margin-top: 80px;
  margin-bottom: 60px;
  width: 100%;
  max-width: 1000px;
  background: rgba(255,255,255,0.95);
  border-radius: 24px;
  box-shadow: 0 4px 18px rgba(85, 107, 47, 0.10);
  padding: 2.5rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-left: auto;
  margin-right: auto;
  overflow-x: auto;
}
.perfil-content h1 {
  font-size: 2.1rem;
  color: #556B2F;
  margin-bottom: 1.5rem;
  font-weight: 600;
  text-align: center;
}
.perfil-table-wrapper {
  width: 100%;
  overflow-x: auto;
}
.perfil-table {
  min-width: 600px;
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px #bdb76b22;
}
.perfil-table th,
.perfil-table td {
  padding: 1rem;
  border-bottom: 1px solid #e0e0b0;
  text-align: left;
  font-size: 1rem;
  vertical-align: middle; 
}
.perfil-table th {
  background: #f8f8e7;
  color: #556B2F;
  font-weight: 600;
}
.perfil-table tr:last-child td {
  border-bottom: none;
}
.edit-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.2rem;
  color: #556B2F;
  transition: color 0.2s;
  padding: 0.2rem 0.5rem;
}
.edit-btn:hover {
  color: #FFD700;
}
/* Estilos para el campo de edición */
.edit-input {
  width: 90%;
  padding: 0.3rem 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: 'Poppins', sans-serif;
  font-size: 1rem;
}
.card-list {
  list-style: disc inside;
  padding-left: 1rem;
  margin: 0;
}
.edit-hint {
  font-size: 0.85rem;
  color: #c0392b;
  margin-top: 0.5rem;
}
.btn-cancelar {
  background: #f4f4f4;
  color: #333;
  border: 1px solid #ccc;
  border-radius: 6px;
  padding: 0.6rem 1.2rem;
  font-size: 0.9rem;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
  margin-top: 1.5rem;
}
.btn-cancelar:hover {
  background: #e0e0e0;
}
</style>