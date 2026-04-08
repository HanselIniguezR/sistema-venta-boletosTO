<template>
  <div class="teatro-page">
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
      <div class="teatro-layout">
        <section class="image-frame" aria-live="polite">
          <div class="image-box">
            <button class="img-nav left" @click="prevImage" aria-label="Imagen anterior">‹</button>
            <img :src="displayedImage" alt="Galería de Obras y Presentaciones Teatrales" /> 
            <button class="img-nav right" @click="nextImage" aria-label="Siguiente imagen">›</button>
          </div>

          <div class="image-caption">
            <h2>Obras Teatrales Destacadas</h2>
            <p class="muted">Galería en Rotación ({{ SLIDE_INTERVAL / 1000 }} segundos)</p>
            <p class="desc">Descubre las mejores producciones de nuestros teatros asociados. La imagen rota automáticamente.</p>
          </div>
        </section>

        <aside class="theaters-panel">
          <h3>Buscar / Seleccionar Teatro</h3>

          <div class="search-wrap">
            <input
              type="text"
              v-model="searchQuery"
              placeholder="Busca un teatro..."
              @input="onSearchInput"
              @focus="showSuggestions = true"
              aria-label="Buscar teatro"
            />
            <ul v-if="showSuggestions && filteredTheaters.length" class="suggestions">
              <li
                v-for="t in filteredTheaters"
                :key="t.id"
                @click="onSuggestionClick(t)"
                class="suggestion-item"
              >
                {{ t.name }} — {{ t.location }}
              </li>
            </ul>
          </div>

          <div class="theater-cards">
            <article class="theater-card" v-if="selectedTheater">
              <header class="theater-header">
                <div class="theater-title">{{ selectedTheater.name }}</div>
                <div class="theater-sub">{{ selectedTheater.location }}</div>
              </header>

              <div class="selectors">
                <label>
                  Categoría de Boleto:
                  <select v-model="selectedTicketCategory">
                    <option v-for="(price, category) in ticketCategories" :key="category" :value="category">
                      {{ category }} (${{ price }})
                    </option>
                  </select>
                </label>
              </div>
              
              <div class="price-info">
                <strong>Precio Total:</strong> ${{ totalPrice.toFixed(2) }}
              </div>

              <div class="selectors">
                <label>
                  Obra:
                  <select v-model="selectedPlayIdByTheater[selectedTheater.id]" @change="resetTimeFor(selectedTheater.id)">
                    <option v-for="p in selectedTheater.plays" :key="p.id" :value="p.id">{{ p.title }}</option>
                  </select>
                </label>
              </div>

              <div class="time-list">
                <label>
                  Horario:
                  <select v-model="selectedTimeByTheater[selectedTheater.id]">
                    <option v-for="time in timesFor(selectedTheater.id)" :key="time" :value="time">{{ time }}</option>
                  </select>
                </label>
              </div>

              <div class="date-picker">
                <label>
                  Fecha:
                  <input
                    type="date"
                    :min="minDate"
                    :max="maxDate"
                    v-model="selectedDateByTheater[selectedTheater.id]"
                    @change="onDateChange(selectedTheater.id)"
                  />
                </label>
                <p class="date-hint" v-if="isCorrectionActive(selectedTheater.id)">
                  ¡Atención! El <strong>{{ correctedDateForTheater[selectedTheater.id] }}</strong> es feriado y no hay eventos disponibles. Se ha seleccionado automáticamente la próxima fecha hábil: <strong>{{ selectedDateByTheater[selectedTheater.id] || '—' }}</strong>
                </p>
              </div>

              <div class="ticket-count">
                <label>
                  Boletos:
                  <input type="number" v-model.number="ticketCount" min="1" max="10" />
                </label>
                <p class="hint">Selecciona hasta {{ ticketLimit }} boletos</p>
              </div>
              
              <div class="card-actions">
                <button class="btn btn-primary" @click="buyFor(selectedTheater.id)">Comprar Boletos</button>
              </div>
            </article>
          </div>
        </aside>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, reactive, onMounted, onBeforeUnmount, watch } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

// Lógica para el menú expandible/colapsable
const isExpanded = ref(false);

function expandMenu() {
  isExpanded.value = true;
}
function collapseMenu() {
  isExpanded.value = false;
}

// Configuración de Categorías de Boletos
const ticketCategories = {
  'Palco': 500.00,
  'Luneta': 350.00,
  'Galería': 200.00,
};
const selectedTicketCategory = ref('Luneta');

// Data: Theaters and Plays (ORIGINAL)
const theaters = [
  {
    id: 'theaterA',
    name: 'Teatro Principal',
    location: 'Centro',
    plays: [
      { id: 'A1', title: 'Romeo y Julieta', company: 'Compañía Estelar', description: 'Drama clásico en nueva producción.', images: ['/teatro1.jpg','/teatro1-2.jpg','/teatro1-3.jpg'], times: ['18:00','21:30'] },
      { id: 'A2', title: 'El Rey León', company: 'Musical SA', description: 'Musical familiar y espectacular.', images: ['/teatro1-2.jpg','/teatro1.jpg','/teatro1-3.jpg'], times: ['19:00','22:00'] },
      { id: 'A3', title: 'Alex Fernández', company: 'Estelar Shows', description: 'Concierto teatro presentación especial.', images: ['/teatro1-3.jpg','/teatro1.jpg','/teatro1-2.jpg'], times: ['17:30','20:00'] },
    ],
  },
  {
    id: 'theaterB',
    name: 'Sala Luminosa',
    location: 'Norte',
    plays: [
      { id: 'B1', title: 'El fantasma de la ópera', company: 'Clásicos SA', description: 'Obra clásica y romántica.', images: ['/teatro2.jpg','/teatro2-2.jpg','/teatro2-3.jpg'], times: ['18:00','20:00'] },
      { id: 'B2', title: 'Les Misérables', company: 'Gran Coro', description: 'Musical épico.', images: ['/teatro2-2.jpg','/teatro2.jpg','/teatro2-3.jpg'], times: ['19:30','22:00'] },
      { id: 'B3', title: 'La pension', company: 'Comedia Teatro', description: 'Comedia dramática ágil.', images: ['/teatro2-3.jpg','/teatro2.jpg','/teatro2-2.jpg'], times: ['17:00','19:00'] },
    ],
  },
  {
    id: 'theaterC',
    name: 'Auditorio Norte',
    location: 'Este',
    plays: [
      { id: 'C1', title: 'Don Juan Tenorio', company: 'Clásicos', description: 'Clásico teatral de tradición.', images: ['/teatro3.jpg','/teatro3-2.jpg','/teatro3-3.jpg'], times: ['18:30','21:00'] },
      { id: 'C2', title: 'Mamma Mia!', company: 'Musicales', description: 'Musical alegre con éxitos.', images: ['/teatro3-2.jpg','/teatro3.jpg','/teatro3-3.jpg'], times: ['19:00','21:45'] },
      { id: 'C3', title: 'Platanito show', company: 'Variedades', description: 'Show cómico familiar.', images: ['/teatro3-3.jpg','/teatro3.jpg','/teatro3-2.jpg'], times: ['17:45','20:15'] },
    ],
  },
];

// HOLIDAYS (festivos)
const holidays = [
  '2025-01-01',
  '2025-05-01',
  '2025-12-25',
  '2025-11-01',
];
const holidaysSet = new Set(holidays);

// utilidades de fecha 
function formatDateYMD(d) {
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2,'0');
  const dd = String(d.getDate()).padStart(2,'0');
  return `${yyyy}-${mm}-${dd}`;
}
function parseYMD(s) {
  if (!s) return null;
  const [y,m,d] = s.split('-').map(Number);
  return new Date(y,m-1,d);
}

// estado 
const selectedTheaterId = ref(theaters[0].id);
const selectedPlayIdByTheater = reactive({});
const selectedTimeByTheater = reactive({});
const selectedDateByTheater = reactive({});
const correctedDateForTheater = reactive({});

theaters.forEach(t => {
  selectedPlayIdByTheater[t.id] = t.plays[0].id;
  selectedTimeByTheater[t.id] = t.plays[0].times[0];
  selectedDateByTheater[t.id] = findNextAvailableDate(formatDateYMD(new Date()));
  correctedDateForTheater[t.id] = null;
});

// computed helpers 
const selectedTheater = computed(() => theaters.find(t => t.id === selectedTheaterId.value) || theaters[0]);
const currentPlay = computed(() => {
  const t = selectedTheater.value;
  return t.plays.find(p => p.id === selectedPlayIdByTheater[t.id]) || t.plays[0];
});

// Cálculo del precio total
const totalPrice = computed(() => {
  const price = ticketCategories[selectedTicketCategory.value] || 0;
  return price * ticketCount.value;
});

// Función para mostrar la advertencia de corrección
function isCorrectionActive(theaterId) {
  return !!correctedDateForTheater[theaterId];
}

// --------------------------------------------------------------------------
// LÓGICA DEL SLIDESHOW INDEPENDIENTE (CON TUS IMÁGENES Y 12 SEG)
// --------------------------------------------------------------------------

// 1. Data de imágenes genéricas (las que tú proporcionaste)
const genericImages = [
    '/RLteatro.jpg',
    '/FOteatro.jpg',
    '/LPteatro.jpg',
    '/DJTteatro.jpg',
    '/RYJteatro.jpg',
    '/CASteatro.jpg'
];

// La imagen mostrada ahora sigue la lista genérica
const displayedImage = ref(genericImages[0]);
const SLIDE_INTERVAL = 12000; // 12 segundos

function rotateImage() {
  const imgs = genericImages;
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx + 1) % imgs.length;
  displayedImage.value = imgs[idx];
}

function prevImage() {
  const imgs = genericImages;
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx - 1 + imgs.length) % imgs.length;
  displayedImage.value = imgs[idx];
}

function nextImage() {
  const imgs = genericImages;
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx + 1) % imgs.length;
  displayedImage.value = imgs[idx];
}

let slideshowTimer = null;
onMounted(() => {
  slideshowTimer = setInterval(rotateImage, SLIDE_INTERVAL);
});
onBeforeUnmount(() => {
  if (slideshowTimer) clearInterval(slideshowTimer);
});

// --------------------------------------------------------------------------
// FIN LÓGICA DEL SLIDESHOW
// --------------------------------------------------------------------------


// buscador / sugerencias 
const searchQuery = ref('');
const showSuggestions = ref(false);
const filteredTheaters = computed(() => {
  const q = (searchQuery.value || '').trim().toLowerCase();
  if (!q) return theaters;
  return theaters.filter(t => `${t.name} ${t.location}`.toLowerCase().includes(q));
});
function onSearchInput() {
  showSuggestions.value = true;
}
function onSuggestionClick(t) {
  selectedTheaterId.value = t.id;
  searchQuery.value = t.name;
  showSuggestions.value = false;
  if (!selectedPlayIdByTheater[t.id]) {
    selectedPlayIdByTheater[t.id] = t.plays[0].id;
    selectedTimeByTheater[t.id] = t.plays[0].times[0];
  }
}

// tickets 
const ticketLimit = 10;
const ticketCount = ref(1);

watch(ticketCount, (n) => {
  if (n < 1) ticketCount.value = 1;
  if (n > ticketLimit) ticketCount.value = ticketLimit;
});

// date helpers
function isHolidayYMD(ymd) { return holidaysSet.has(ymd); }
function findNextAvailableDate(startYMD) {
  let d = parseYMD(startYMD) || new Date();
  if (!(d instanceof Date) || isNaN(d)) d = new Date();
  for (let i=0;i<365;i++) {
    const ymd = formatDateYMD(d);
    if (!isHolidayYMD(ymd)) return ymd;
    d.setDate(d.getDate() + 1);
  }
  return formatDateYMD(new Date());
}

const minDate = formatDateYMD(new Date());
const maxDate = formatDateYMD(new Date(new Date().getFullYear() + 1, 11, 31));

// Lógica de corrección de fecha
function onDateChange(theaterId) {
  const selected = selectedDateByTheater[theaterId];
  correctedDateForTheater[theaterId] = null;

  if (!selected) return;
  if (isHolidayYMD(selected)) {
    const next = findNextAvailableDate(selected);
    if (selected !== next) {
      correctedDateForTheater[theaterId] = selected;
      selectedDateByTheater[theaterId] = next;
    }
  }
}

// compra
function timesFor(theaterId) {
  const t = theaters.find(x => x.id === theaterId);
  if (!t) return [];
  const pid = selectedPlayIdByTheater[theaterId];
  const p = t.plays.find(x => x.id === pid);
  return p ? p.times : (t.plays[0]?.times || []);
}
function resetTimeFor(theaterId) {
  const t = theaters.find(x => x.id === theaterId);
  const pid = selectedPlayIdByTheater[theaterId];
  const p = t.plays.find(x => x.id === pid);
  selectedTimeByTheater[theaterId] = p ? p.times[0] : (t.plays[0].times[0]);
}

function buyFor(theaterId) {
  const date = selectedDateByTheater[theaterId];
  if (!date) { alert('Debe seleccionar una fecha válida.'); return; }
  
  // DOBLE VERIFICACIÓN DE DÍA FESTIVO
  if (isHolidayYMD(date)) { 
    alert('ERROR: La fecha seleccionada es feriado y NO está disponible para compra. Por favor, seleccione una fecha hábil. (El sistema intentó auto-corregir al día hábil más cercano).'); 
    const next = findNextAvailableDate(date);
    selectedDateByTheater[theaterId] = next;
    return; 
  }

  const category = selectedTicketCategory.value;
  const totalAmount = totalPrice.value;

  const t = theaters.find(x => x.id === theaterId);
  const play = t.plays.find(p => p.id === selectedPlayIdByTheater[theaterId]);
  const time = selectedTimeByTheater[theaterId];
  
  // Si el usuario no está autenticado, guardar intención y redirigir a SignIn
  const isAuth = !!(localStorage.getItem('isAuthenticated') || sessionStorage.getItem('isAuthenticated'));
  const purchasePayload = {
    type: 'TEATRO',
    play: play.id,
    theater: t.id,
    time,
    date,
    category,
    total: totalAmount,
    tickets: ticketCount.value
  };
  if (!isAuth) {
    sessionStorage.setItem('pendingPurchase', JSON.stringify(purchasePayload));
    router.push('/signin');
    return;
  }

  // Usuario autenticado: ir a /pay con la información en query
  router.push({
    path: '/pay',
    query: {
      play: play.id,
      theater: t.id,
      time,
      date,
      ticketType: category,
      total: totalAmount,
      tickets: ticketCount.value
    }
  });
}

// logout (ORIGINAL)
function logout() {
  try { localStorage.setItem('isAuthenticated', 'false'); } catch (e) {}
  try { localStorage.removeItem('token'); } catch (e) {}
  try { localStorage.removeItem('user'); } catch (e) {}
  try { sessionStorage.removeItem('pendingPurchase'); } catch (e) {}
  try { sessionStorage.clear(); } catch (e) {}
  try { router.replace('/'); } catch (e) { router.push('/'); }
}
</script>

<style scoped>
.teatro-page {
  min-height: 100vh;
  width: 100%;
  position: relative;
  font-family: 'Poppins', sans-serif;
  background: #FFFDD0;
  display: flex;
}

/* monograma */
.monogram-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  opacity: 0.06;
  background-image: url('/monogram.png');
  background-repeat: repeat;
  background-size: 120px 120px;
  filter: grayscale(1) brightness(1.2);
}

/* -------------------------------------------------------------------------- */
/* SIDEBAR EXPANDIBLE */
/* -------------------------------------------------------------------------- */
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
  margin: 0 0 2rem 0; 
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

.menu-item:hover, .menu-item.active, .router-link-active.menu-item { 
  background: #445522; 
  color: #FFD700; 
}

.text { 
  white-space: nowrap; 
  overflow: hidden; 
  text-overflow: ellipsis; 
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
.btn-logout:hover { background: #FFD700; color: #445522; }
/* -------------------------------------------------------------------------- */
/* FIN DE ESTILOS SIDEBAR */
/* -------------------------------------------------------------------------- */

/* Layout principal */
.main-content { flex: 1; padding: 2.5rem; display: flex; align-items: flex-start; justify-content: center; z-index: 1; }
.teatro-layout { display: grid; grid-template-columns: 1fr 420px; gap: 1.5rem; width: 100%; max-width: 1200px; }

/* Image frame */
.image-frame { display:flex; flex-direction:column; gap:1rem; align-items:center; }
.image-box { 
  width:100%; 
  max-width:720px; 
  height:580px; /* <--- ALTO MODIFICADO A LO LARGO */
  border-radius:16px; 
  overflow:hidden; 
  background:#f5f5f5; 
  box-shadow:0 8px 24px rgba(0,0,0,0.06); 
  position:relative; 
  display:flex; 
  align-items:center; 
  justify-content:center; 
}
.image-box img { width:100%; height:100%; object-fit:cover; display:block; }
.img-nav { position:absolute; top:50%; transform:translateY(-50%); background:rgba(0,0,0,0.35); color:#fff; border:none; padding:0.5rem 0.9rem; border-radius:6px; cursor:pointer; font-size:1.6rem; }
.img-nav.left { left:12px; } .img-nav.right { right:12px; }
.image-caption { width:100%; max-width:720px; text-align:left; }
.image-caption h2 { margin:0; color:#223; font-size:1.25rem; }
.image-caption .muted { color:#666; margin:0.25rem 0; }
.image-caption .desc { color:#444; margin-top:0.5rem; }

/* Panel derecho */
.theaters-panel { background:transparent; }
.theaters-panel h3 { margin:0 0 0.75rem 0; color:#223; }

/* Buscador */
.search-wrap { position:relative; margin-bottom:0.75rem; }
.search-wrap input { width:100%; padding:0.5rem 0.6rem; border-radius:6px; border:1px solid #e6e6e6; font-family:'Poppins',sans-serif; }
.suggestions { position:absolute; top:calc(100% + 6px); left:0; right:0; background:#fff; border:1px solid #e6e6e6; border-radius:6px; max-height:180px; overflow:auto; z-index:10; box-shadow:0 8px 18px rgba(0,0,0,0.06); margin:0; padding:0; list-style:none; }
.suggestion-item { padding:0.55rem 0.75rem; cursor:pointer; }
.suggestion-item:hover { background:#f6f6f6; }

/* Card */
.theater-cards { display:flex; flex-direction:column; gap:1rem; }
.theater-card { background:rgba(255,255,255,0.98); border-radius:12px; padding:0.9rem; box-shadow:0 8px 18px rgba(0,0,0,0.06); border:1px solid rgba(0,0,0,0.03); }
.theater-header { display:flex; flex-direction:column; gap:0.125rem; margin-bottom:0.5rem; }
.theater-title { font-weight:600; color:#223; } .theater-sub { font-size:0.85rem; color:#666; }

/* Selectores de Categoría */
.selectors label { display: block; margin-bottom: 0.5rem; font-weight: 500; color: #444; }
.selectors select, .time-list select { width: 100%; padding: 0.4rem 0.5rem; border-radius: 6px; border: 1px solid #e6e6e6; margin-top: 0.2rem; font-family: 'Poppins', sans-serif; }
.time-list { margin-top:0.8rem; }
.price-info { 
  margin:0.5rem 0 0.8rem 0; 
  font-size:1.1rem; 
  color:#445522; 
  font-weight: 600;
} 

/* date picker */
.date-picker { margin:0.4rem 0 0.8rem 0; }
.date-picker input[type="date"] { padding:0.4rem 0.5rem; border-radius:6px; border:1px solid #e6e6e6; }
.date-hint { 
  margin:0.35rem 0 0; 
  color:#c0392b; 
  font-size:0.9rem; 
  background-color: #fcebeb; 
  padding: 0.4rem;
  border-radius: 4px;
  border: 1px solid #c0392b;
}

/* tickets */
.ticket-count { margin:0.5rem 0; }
.ticket-count input { width:80px; padding:0.3rem; border-radius:6px; border:1px solid #e6e6e6; }

.card-actions { display:flex; gap:0.5rem; margin-top:0.6rem; justify-content:flex-end; }
.btn { padding:0.55rem 0.9rem; border-radius:8px; cursor:pointer; border:1px solid transparent; font-size:0.95rem; }
.btn-primary { background:#556B2F; color:#fff; border-color:#556B2F; }

/* Responsive */
@media (max-width: 1100px) {
  .teatro-layout { grid-template-columns: 1fr; }
  .theaters-panel { order: 2; margin-top: 1rem; }
  .image-frame { order: 1; }
  .image-box { height: 400px; max-width: 100%; } /* Ajuste responsive */
}

@media (max-width: 480px) {
  /* En móvil, el sidebar queda expandido a 220px */
  .sidebar { 
    width: 220px; 
    min-width: 220px; 
    align-items: flex-start; 
    padding-left: 1.25rem;
    display: flex; 
  }
  .sidebar h2 { display: block; padding-left: 0; margin-bottom: 1rem; }
  .menu-item { 
    font-size: 0.98rem; 
    padding: 0.7rem 1rem; 
    border-bottom: none;
  }
  .btn-logout { display: block; width: calc(100% - 2.5rem); margin-top: 1.25rem; }
  .image-box { height: 280px; } /* Ajuste responsive */
  .card-actions { justify-content: center; }
}
</style>