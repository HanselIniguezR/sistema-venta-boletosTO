<template>
  <div class="museo-page">
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

      <button @click="logout" class="btn-logout" v-if="isExpanded">Cerrar sesión</button>
    </aside>

    <main class="main-content">
      <div class="museo-layout">
        
        <section class="image-frame" aria-live="polite">
          <div class="image-box">
            <button class="img-nav left" @click="prevImage" aria-label="Imagen anterior">‹</button>
            <img :src="displayedImage" :alt="currentExhibitTitle" /> 
            <button class="img-nav right" @click="nextImage" aria-label="Siguiente imagen">›</button>
          </div>

          <div class="image-caption">
            <h2>{{ currentMuseum.name }}</h2>
            <p class="muted">{{ currentMuseum.location }}</p>
            <p class="desc">{{ currentExhibitTitle }}</p>
          </div>
        </section>

        <aside class="selection-panel">
          <h3>Buscar / Seleccionar Museo</h3>

          <div class="search-wrap">
            <input
              type="text"
              v-model="searchQuery"
              placeholder="Busca un museo..."
              @input="onSearchInput"
              @focus="showSuggestions = true"
              aria-label="Buscar museo"
            />
            <ul v-if="showSuggestions && filteredMuseums.length" class="suggestions">
              <li
                v-for="m in filteredMuseums"
                :key="m.id"
                @click="onSuggestionClick(m)"
                class="suggestion-item"
              >
                {{ m.name }} — {{ m.location }}
              </li>
            </ul>
          </div>

          <div class="museum-cards">
            <article class="museum-card" v-if="currentMuseum.id !== 'default-museum'"> 
              <header class="museum-header">
                <div class="museum-title">{{ currentMuseum.name }}</div>
                <div class="museum-sub">{{ currentMuseum.location }}</div>
              </header>
              
              <div class="date-picker">
                <label>
                  Fecha de Visita:
                  <input
                    type="date"
                    :min="minDate"
                    :max="maxDate"
                    v-model="selectedDate"
                  />
                </label>
              </div>

              <div class="ticket-count">
                <label>
                  Boletos (Máx 5):
                  <input type="number" v-model.number="ticketCount" min="1" :max="ticketLimit" />
                </label>
                <p class="hint">Máximo {{ ticketLimit }} boletos por transacción.</p>
              </div>
              
              <div class="price-info">
                <strong>Precio Unitario:</strong> ${{ currentMuseum.price.toFixed(2) }}
              </div>
              <div class="price-info total">
                <strong>Precio Total:</strong> ${{ totalPrice.toFixed(2) }}
              </div>

              <div class="card-actions">
                <button class="btn btn-primary" @click="buyTickets">Comprar Boletos</button>
              </div>
            </article>
          </div>
        </aside>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

// Estado del menú (mantenido)
const isExpanded = ref(false);
function expandMenu() { isExpanded.value = true; }
function collapseMenu() { isExpanded.value = false; }
function logout() { 
  try { localStorage.removeItem('isAuthenticated'); localStorage.removeItem('token'); localStorage.removeItem('user'); sessionStorage.clear(); } catch (e) {}
  router.push('/signin');
}

// --- DATOS FIJOS DEL MUSEO (Sintaxis verificada) ---
const MUSEUMS = [
  { 
    id: 'museo_arte_moderno', 
    name: 'Museo de Arte Moderno', 
    location: 'Centro Histórico', 
    description: 'Colección de arte moderno y contemporáneo.',
    images: ['/VGMuseo.jpg', '/TBMuseo.avif', '/DibujoMuseo.jpg', '/arte_moderno_1.jpg', '/arte_moderno_2.jpg', '/minimalista_1.jpg'], 
    price: 80.00
  },
  { 
    id: 'museo_historia_natural', 
    name: 'Museo de Historia Natural', 
    location: 'Chapultepec', 
    description: 'Exhibiciones de dinosaurios y ecosistemas.',
    images: ['/VGMuseo.jpg', '/TBMuseo.avif', '/DibujoMuseo.jpg', '/dinos_1.jpg', '/dinos_2.jpg', '/dinos_3.jpg'], 
    price: 90.00
  },
  { 
    id: 'museo_ciencia', 
    name: 'Universum - Museo de la Ciencia', 
    location: 'Coyoacán', 
    description: 'Exploración interactiva del cosmos y la ciencia.',
    images: ['/VGMuseo.jpg', '/TBMuseo.avif', '/DibujoMuseo.jpg', '/cosmos_1.jpg', '/cosmos_2.jpg'], 
    price: 75.00
  },
  { 
    id: 'museo_antropologia', 
    name: 'Museo de Antropología', 
    location: 'Reforma', 
    description: 'Gran colección de culturas mesoamericanas.',
    images: ['/VGMuseo.jpg', '/TBMuseo.avif', '/DibujoMuseo.jpg', '/antropo_1.jpg', '/antropo_2.jpg'], 
    price: 100.00
  },
  { 
    id: 'museo_palacio', 
    name: 'Palacio de Bellas Artes', 
    location: 'Alameda Central', 
    description: 'Famoso por sus murales y eventos culturales.',
    images: ['/VGMuseo.jpg', '/TBMuseo.avif', '/DibujoMuseo.jpg', '/murales_1.jpg', '/murales_2.jpg'], 
    price: 120.00
  },
];

// --- INITIALIZATION GUARD ---
const DEFAULT_MUSEUM = { id: 'default-museum', name: 'Museo no disponible', location: 'N/A', description: 'Selecciona un museo disponible.', images: ['/default.jpg'], price: 0.00 };

const initialMuseum = MUSEUMS[0] || DEFAULT_MUSEUM;

// --- ESTADOS DE SELECCIÓN ---
const selectedMuseumId = ref(initialMuseum.id);
const ticketLimit = 5;
const ticketCount = ref(1);

// Lógica de fecha (Mantenida)
function formatDateYMD(d) {
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2,'0');
  const dd = String(d.getDate()).padStart(2,'0');
  return `${yyyy}-${mm}-${dd}`;
}
const today = new Date();
const minDate = formatDateYMD(today);
const maxDateObj = new Date(today.getFullYear(), today.getMonth() + 3, today.getDate());
const maxDate = formatDateYMD(maxDateObj); 
const selectedDate = ref(minDate);


// --- PROPIEDADES COMPUTADAS ---
const currentMuseum = computed(() => MUSEUMS.find(m => m.id === selectedMuseumId.value) || DEFAULT_MUSEUM);

// Para el carrusel, ahora solo necesitamos las imágenes del museo actual
const currentMuseumImages = computed(() => currentMuseum.value.images || [DEFAULT_MUSEUM.images[0]]);

// Para el título de la descripción, usamos la descripción general
const currentExhibitTitle = computed(() => currentMuseum.value.description);

const totalPrice = computed(() => currentMuseum.value.price * ticketCount.value);

// --- LÓGICA DE BÚSQUEDA ---
const searchQuery = ref('');
const showSuggestions = ref(false);

const filteredMuseums = computed(() => {
  const q = (searchQuery.value || '').trim().toLowerCase();
  if (!q) return MUSEUMS;
  return MUSEUMS.filter(m => `${m.name} ${m.location}`.toLowerCase().includes(q));
});

function onSearchInput() {
  showSuggestions.value = true;
}

function onSuggestionClick(museum) {
  selectedMuseumId.value = museum.id;
  searchQuery.value = museum.name;
  showSuggestions.value = false;
  // Reiniciar el carrusel y la imagen al seleccionar un nuevo museo
  displayedImage.value = museum.images?.[0] || DEFAULT_MUSEUM.images[0];
  resetSlideshow();
}

// --- LÓGICA DEL CARRUSEL (Actualizada para usar imágenes de MUSEO) ---
const displayedImage = ref(currentMuseumImages.value[0] || DEFAULT_MUSEUM.images[0]);
let slideshowTimer = null;
const SLIDE_INTERVAL = 10000;

function prevImage() {
  const imgs = currentMuseumImages.value;
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx - 1 + imgs.length) % imgs.length;
  displayedImage.value = imgs[idx];
}
function nextImage() {
  const imgs = currentMuseumImages.value;
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx + 1) % imgs.length;
  displayedImage.value = imgs[idx];
}
function rotateImage() {
  nextImage();
}
function resetSlideshow() {
    if (slideshowTimer) clearInterval(slideshowTimer);
    slideshowTimer = setInterval(rotateImage, SLIDE_INTERVAL);
}

onMounted(() => {
  resetSlideshow();
});

onBeforeUnmount(() => {
  if (slideshowTimer) clearInterval(slideshowTimer);
});

// Observar el museo actual para actualizar el carrusel
watch(currentMuseum, (newMuseum) => {
  displayedImage.value = newMuseum.images?.[0] || DEFAULT_MUSEUM.images[0];
  resetSlideshow();
}, { deep: true });

// --- LÓGICA DE COMPRA ---
function buyTickets() {
  if (ticketCount.value < 1 || ticketCount.value > ticketLimit) {
    alert(`Por favor, selecciona una cantidad de boletos válida (1 a ${ticketLimit}).`);
    return;
  }
  if (!selectedDate.value) {
    alert('Debes seleccionar una fecha de visita.');
    return;
  }
  
  // Redirigir a la vista de pago con los parámetros necesarios
  router.push({
    path: `/purchase/museo/${selectedMuseumId.value}`, // Usando la ruta dinámica
    query: {
      museum: selectedMuseumId.value,
      date: selectedDate.value,
      count: ticketCount.value,
      total: totalPrice.value.toFixed(2),
      id: selectedMuseumId.value // Necesario para que la vista de pago obtenga el título
    }
  });
}

watch(ticketCount, (n) => {
    if (n < 1) ticketCount.value = 1;
    if (n > ticketLimit) ticketCount.value = ticketLimit; // Corregida la lógica
});
</script>

<style scoped>
/* -------------------------------------------------------------------------- */
/* BASE: Mantiene los estilos del sidebar y fondo del componente Cine/Home */
/* -------------------------------------------------------------------------- */

.museo-page {
  display: flex;
  height: 100vh;
  background: #FFFDD0;
  font-family: 'Poppins', sans-serif;
  position: relative;
}
.monogram-bg {
  position: fixed; inset: 0; z-index: 0; pointer-events: none; opacity: 0.07;
  background-image: url('/monogram.png'); background-repeat: repeat;
  background-size: 120px 120px; background-position: center; filter: grayscale(1) brightness(1.2);
}
.main-content {
  flex: 1;
  padding: 2.5rem;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  z-index: 1;
  overflow-y: auto;
}
/* Estilos del Sidebar */
.sidebar {
  background: #556B2F; color: #fff; width: 70px; min-width: 70px; transition: width 0.3s;
  padding-top: 40px; padding-bottom: 40px; box-shadow: 2px 0 8px rgba(85, 107, 47, 0.08);
  display: flex; flex-direction: column; align-items: center; position: sticky; top: 0; z-index: 2;
  height: 100vh;
}
.sidebar.expanded { width: 220px; min-width: 220px; align-items: flex-start; }
.sidebar h2 { margin-bottom: 2rem; font-size: 1.1rem; font-weight: 500; letter-spacing: 1px; color: #fff; text-align: left; width: 100%; padding-left: 2rem; display: none; }
.sidebar.expanded h2 { display: block; }
.sidebar ul { list-style: none; padding: 0; margin: 0; width: 100%; }
.menu-item { display: flex; align-items: center; width: 100%; padding: 1.1rem 0.5rem; color: #fff; text-decoration: none; font-size: 0; border: none; background: none; border-radius: 0; border-bottom: 1px solid #6b8e23; transition: background 0.2s, color 0.2s, font-size 0.3s, padding 0.3s; cursor: pointer; font-weight: 500; letter-spacing: 0.5px; text-align: left; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.menu-item:last-child { border-bottom: none; }
.sidebar.expanded .menu-item { font-size: 1.08rem; padding: 1.1rem 2rem; }
.menu-item:hover, .menu-item.active, .router-link-active.menu-item { background: #445522; color: #FFD700; }
.btn-logout { background: #fff; color: #556B2F; border: none; border-radius: 6px; padding: 0.7rem 1.5rem; font-size: 1rem; font-family: 'Poppins', sans-serif; font-weight: 500; cursor: pointer; transition: background 0.2s; margin-top: 2rem; width: 80%; align-self: center; display: none; }
.sidebar.expanded .btn-logout { display: block; }


/* -------------------------------------------------------------------------- */
/* ESTILOS ESPECÍFICOS DE MUSEO (Tomados y adaptados de Cine) */
/* -------------------------------------------------------------------------- */

.museo-layout {
  display: grid;
  grid-template-columns: 1fr 420px;
  gap: 1.5rem;
  width: 100%;
  max-width: 1200px;
}
/* SECCIÓN IZQUIERDA: CARRUSEL */
.image-frame { display:flex; flex-direction:column; gap:1rem; align-items:center; }
.image-box { width:100%; max-width:720px; height:420px; border-radius:16px; overflow:hidden; background:#f5f5f5; box-shadow:0 8px 24px rgba(0,0,0,0.06); position:relative; display:flex; align-items:center; justify-content:center; }
.image-box img { width:100%; height:100%; object-fit:cover; display:block; }
.img-nav { position:absolute; top:50%; transform:translateY(-50%); background:rgba(0,0,0,0.35); color:#fff; border:none; padding:0.5rem 0.9rem; border-radius:6px; cursor:pointer; font-size:1.6rem; }
.img-nav.left { left:12px; } .img-nav.right { right:12px; }
.image-caption { width:100%; max-width:720px; text-align:left; }
.image-caption h2 { margin:0; color:#223; font-size:1.25rem; }
.image-caption .muted { color:#666; margin:0.25rem 0; }
.image-caption .desc { color:#444; margin-top:0.5rem; }

/* SECCIÓN DERECHA: SELECCIÓN */
.selection-panel { background:transparent; }
.selection-panel h3 { margin:0 0 0.75rem 0; color:#223; }
.search-wrap { position:relative; margin-bottom:0.75rem; }
.search-wrap input { width:100%; padding:0.5rem 0.6rem; border-radius:6px; border:1px solid #e6e6e6; font-family:'Poppins',sans-serif; }
.suggestions { position:absolute; top:calc(100% + 6px); left:0; right:0; background:#fff; border:1px solid #e6e6e6; border-radius:6px; max-height:180px; overflow:auto; z-index:10; box-shadow:0 8px 18px rgba(0,0,0,0.06); margin:0; padding:0; list-style:none; }
.suggestion-item { padding:0.55rem 0.75rem; cursor:pointer; }
.suggestion-item:hover { background:#f6f6f6; }
.museum-cards { display:flex; flex-direction:column; gap:1rem; }
.museum-card { background:rgba(255,255,255,0.98); border-radius:12px; padding:0.9rem; box-shadow:0 8px 18px rgba(0,0,0,0.06); border:1px solid rgba(0,0,0,0.03); }
.museum-header { display:flex; flex-direction:column; gap:0.125rem; margin-bottom:0.5rem; }
.museum-title { font-weight:600; color:#223; } .museum-sub { font-size:0.85rem; color:#666; }

.price-info { margin:0.5rem 0 0.2rem 0; font-size:1rem; color:#444; }
.price-info.total { margin-top: 0.6rem; font-size:1.15rem; color:#556B2F; font-weight: 600; }

/* Hemos eliminado los estilos de selector y tiempo */
.date-picker { margin:0.8rem 0 0.8rem 0; }
.ticket-count { margin:0.8rem 0; }
.ticket-count input { width:80px; padding:0.3rem; border-radius:6px; border:1px solid #e6e6e6; }
.hint { font-size: 0.85rem; color: #777; margin-top: 0.25rem; }

.card-actions { display:flex; gap:0.5rem; margin-top:1rem; justify-content:flex-end; }
.btn { padding:0.55rem 0.9rem; border-radius:8px; cursor:pointer; border:1px solid transparent; font-size:0.95rem; }
.btn-primary { background:#556B2F; color:#fff; border-color:#556B2F; }

/* -------------------------------------------------------------------------- */
/* Responsive */
/* -------------------------------------------------------------------------- */

@media (max-width: 1100px) {
  .museo-layout { grid-template-columns: 1fr; }
  .selection-panel { order: 2; margin-top: 1rem; }
  .image-frame { order: 1; }
  .image-box { height: 300px; max-width: 100%; }
}

@media (max-width: 480px) {
  .image-box { height: 220px; }
  .card-actions { justify-content: center; }
}
</style>