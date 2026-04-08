<template>
  <div class="cine-page">
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
      <div class="cine-layout">
        <section class="image-frame" aria-live="polite">
          <div class="image-box">
            <button class="img-nav left" @click="prevImage" aria-label="Imagen anterior">‹</button>
            <img :src="displayedImage" :alt="currentMovie.title" />
            <button class="img-nav right" @click="nextImage" aria-label="Siguiente imagen">›</button>
          </div>

          <div class="image-caption">
            <h2>{{ currentMovie.title }}</h2>
            <p class="muted">{{ currentMovie.company }}</p>
            <p class="desc">{{ currentMovie.description }}</p>
          </div>
        </section>

        <aside class="theaters-panel">
          <h3>Buscar / Seleccionar Cine</h3>

          <div class="search-wrap">
            <input
              type="text"
              v-model="searchQuery"
              placeholder="Busca un cine..."
              @input="onSearchInput"
              @focus="showSuggestions = true"
              aria-label="Buscar cine"
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
                  Tipo de Boleto:
                  <select v-model="selectedTicketType">
                    <option v-for="(price, type) in ticketPrices" :key="type" :value="type">
                      {{ type }} (${{ price }})
                    </option>
                  </select>
                </label>
              </div>

              <div class="price-info">
                <strong>Precio Total:</strong> ${{ totalPrice.toFixed(2) }}
              </div>

              <div class="selectors">
                <label>
                  Película:
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
                <p class="date-hint" v-if="isHolidaySelected(selectedTheater.id)">
                  La fecha seleccionada es feriado y no está disponible. Se eligió la próxima fecha disponible: <strong>{{ selectedDateByTheater[selectedTheater.id] || '—' }}</strong>
                </p>
              </div>

              <div class="ticket-count">
                <label>
                  Boletos:
                  <input type="number" v-model.number="ticketCount" min="1" max="10" />
                </label>
                <p class="hint">Selecciona hasta {{ ticketLimit }} asientos</p>
              </div>

              <div class="seat-map-container" v-if="selectedTheater">

                <div class="screen">Pantalla</div>
                <div class="seat-legend">
                  <span class="seat available"></span> Disponible
                  <span class="seat selected"></span> Seleccionado
                </div>

                <div class="seat-grid" role="grid" aria-label="Mapa de asientos">
                  <div
                    v-for="row in seatRows"
                    :key="row"
                    class="seat-row"
                    role="row"
                  >
                    <div class="row-label">{{ row }}</div>
                    <button
                      v-for="col in seatCols"
                      :key="col"
                      :class="['seat', seatClass(row,col)]"
                      :disabled="isSeatDisabled(row,col)"
                      @click.prevent="toggleSeat(row,col)"
                      :aria-pressed="isSeatSelected(row,col)"
                    >
                      {{ row }}{{ col }}
                    </button>
                  </div>
                </div>

                <div class="selected-list">
                  <strong>Seleccionados:</strong>
                  <span v-if="selectedSeats.length === 0"> (ninguno) </span>
                  <span v-for="s in selectedSeats" :key="s" class="sel-item">{{ s }}</span>
                </div>
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

// Nuevo estado de expansión del menú
const isExpanded = ref(false);

function expandMenu() {
  isExpanded.value = true;
}
function collapseMenu() {
  isExpanded.value = false;
}

// Configuración de Precios de Boletos
const ticketPrices = {
  'Tradicional': 95.00,
  'VIP': 150.00,
  'IMAX': 180.00,
};
const selectedTicketType = ref('Tradicional');

// --- DEFINICIONES DE PELÍCULAS (IMÁGENES ACTUALIZADAS) ---
const fixedPlays = [
  {
    id: 'BATMAN',
    title: 'The Batman',
    company: 'Warner Bros.',
    description: 'El Caballero Oscuro se enfrenta a un nuevo acertijo en Gotham.',
    images: ['/TheBatCine.jpg', '/thebatman-2.jpg', '/thebatman-3.jpg'], 
    times: ['15:00','18:30','21:30','00:30']
  },
  {
    id: 'SUPERMAN',
    title: 'Superman',
    company: 'DC Studios',
    description: 'El Hombre de Acero regresa para proteger Metrópolis de nuevas amenazas.',
    images: ['/SupermanCine.webp', '/superman-2.jpg', '/superman-3.jpg'],
    times: ['14:00','16:00','20:00','23:00']
  },
  {
    id: 'SPIDERMAN',
    title: 'Spider-Man: Brand New Day!',
    company: 'Marvel',
    description: 'Una nueva era para el amigable vecino Spider-Man con nuevos villanos.',
    images: ['/SpideyCine.webp', '/spiderman-2.jpg', '/spiderman-3.jpg'],
    times: ['17:00','19:45','22:15','01:30']
  },
];

// Data: Cines y Películas
const theaters = [
  { id: 'cinepolis', name: 'Cinépolis', location: 'Plaza Central', plays: fixedPlays },
  { id: 'cinemex', name: 'Cinemex', location: 'Las Torres', plays: fixedPlays },
  { id: 'cinemark', name: 'Cinemark', location: 'Gran Explanada', plays: fixedPlays },
  { id: 'amc', name: 'AMC Theatres', location: 'Metrópolis', plays: fixedPlays },
];

// HOLIDAYS (festivos)
const holidays = [
  '2025-01-01', '2025-05-01', '2025-12-25', '2025-11-01',
];
const holidaysSet = new Set(holidays);


/* --- LÓGICA DE ESTADOS Y UTILIDADES --- */

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

const selectedTheaterId = ref(theaters[0]?.id);
const selectedPlayIdByTheater = reactive({});
const selectedTimeByTheater = reactive({});
const selectedDateByTheater = reactive({});

// Inicialización de estados
theaters.forEach(t => {
  const defaultPlay = t.plays[0];
  if (defaultPlay) {
    selectedPlayIdByTheater[t.id] = defaultPlay.id;
    selectedTimeByTheater[t.id] = defaultPlay.times[0];
  } else {
    selectedPlayIdByTheater[t.id] = fixedPlays[0].id;
    selectedTimeByTheater[t.id] = fixedPlays[0].times[0];
  }
  selectedDateByTheater[t.id] = findNextAvailableDate(formatDateYMD(new Date()));
});

// Propiedades Computadas
const selectedTheater = computed(() => theaters.find(t => t.id === selectedTheaterId.value) || theaters[0] || null);

const currentMovie = computed(() => {
  const theater = selectedTheater.value;
  if (!theater) return fixedPlays[0];

  const playId = selectedPlayIdByTheater[theater.id] || fixedPlays[0].id;
  return fixedPlays.find(p => p.id === playId) || fixedPlays[0];
});

const totalPrice = computed(() => {
  const price = ticketPrices[selectedTicketType.value] || 0;
  return price * ticketCount.value;
});

// Lógica de Imagen y Slideshow
const displayedImage = ref(currentMovie.value.images?.[0] || '/TheBatCine.jpg'); 

watch(currentMovie, (newMovie) => {
  if (slideshowTimer) clearInterval(slideshowTimer);
  displayedImage.value = newMovie.images?.[0] || '/TheBatCine.jpg'; 
  slideshowTimer = setInterval(rotateImage, SLIDE_INTERVAL);
}, { deep: true });

function prevImage() {
  const imgs = currentMovie.value.images || [displayedImage.value];
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx - 1 + imgs.length) % imgs.length;
  displayedImage.value = imgs[idx];
}
function nextImage() {
  const imgs = currentMovie.value.images || [displayedImage.value];
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx + 1) % imgs.length;
  displayedImage.value = imgs[idx];
}
let slideshowTimer = null;
const SLIDE_INTERVAL = 10000;
function rotateImage() {
  const imgs = currentMovie.value.images || [displayedImage.value];
  let idx = imgs.indexOf(displayedImage.value);
  if (idx === -1) idx = 0;
  idx = (idx + 1) % imgs.length;
  displayedImage.value = imgs[idx];
}
onMounted(() => {
  slideshowTimer = setInterval(rotateImage, SLIDE_INTERVAL);
});
onBeforeUnmount(() => {
  if (slideshowTimer) clearInterval(slideshowTimer);
});

// Lógica de Búsqueda y Selección de Cine
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
}

// Lógica de Asientos y Boletos
const ticketLimit = 10;
const ticketCount = ref(1);
const seatRows = ['A','B','C','D','E','F','G'];
const seatCols = Array.from({length:25}, (_,i) => i+1);
const selectedSeats = ref([]);
watch(selectedTheaterId, () => { selectedSeats.value = []; });
function seatId(row, col) { return `${row}${col}`; }
function isSeatSelected(row, col) { return selectedSeats.value.includes(seatId(row,col)); }
function seatClass(row, col) { return selectedSeats.value.includes(seatId(row,col)) ? 'selected' : 'available'; }
function isSeatDisabled(row, col) {
  const id = seatId(row,col);
  if (selectedSeats.value.length >= ticketCount.value && !selectedSeats.value.includes(id)) return true;
  return false;
}
function toggleSeat(row, col) {
  const id = seatId(row,col);
  const idx = selectedSeats.value.indexOf(id);
  if (idx !== -1) { selectedSeats.value.splice(idx,1); return; }
  if (selectedSeats.value.length < ticketCount.value) selectedSeats.value.push(id);
}
watch(ticketCount, (n) => {
  if (n < 1) ticketCount.value = 1;
  if (n > ticketLimit) ticketCount.value = ticketLimit;
  while (selectedSeats.value.length > ticketCount.value) selectedSeats.value.pop();
});

// Lógica de Fecha y Feriados
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
function onDateChange(theaterId) {
  const selected = selectedDateByTheater[theaterId];
  if (!selected) return;
  if (isHolidayYMD(selected)) {
    const next = findNextAvailableDate(selected);
    selectedDateByTheater[theaterId] = next;
  }
}
function isHolidaySelected(theaterId) {
  const s = selectedDateByTheater[theaterId];
  if (!s) return false;
  return holidaysSet.has(s);
}
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
  selectedTimeByTheater[theaterId] = p ? p.times[0] : (t.plays[0]?.times[0]);
}

// Lógica de Compra
function buyFor(theaterId) {
  const date = selectedDateByTheater[theaterId];
  if (!date) { alert('Debe seleccionar una fecha válida.'); return; }
  if (isHolidayYMD(date)) { alert('La fecha seleccionada es feriado y no está disponible.'); return; }
  if (selectedSeats.value.length !== ticketCount.value) { alert(`Debes seleccionar ${ticketCount.value} asientos. Actualmente: ${selectedSeats.value.length}`); return; }

  const ticketType = selectedTicketType.value;
  const totalAmount = totalPrice.value;

  const t = theaters.find(x => x.id === theaterId);
  const play = fixedPlays.find(p => p.id === selectedPlayIdByTheater[theaterId]);
  const time = selectedTimeByTheater[theaterId];

  if (!play || !t) { alert('Error: Faltan datos de la función o del cine.'); return; }

  // Si el usuario no está autenticado, redirigir a SignIn/Login primero
  const isAuth = !!(localStorage.getItem('isAuthenticated') || sessionStorage.getItem('isAuthenticated'));
  if (!isAuth) {
    // Guardamos la intención de compra en sessionStorage para continuar después del login
    sessionStorage.setItem('pendingPurchase', JSON.stringify({
      play: play.id,
      theater: t.id,
      time: time,
      seats: selectedSeats.value.join(','),
      date: date,
      ticketType: ticketType,
      total: (typeof totalAmount === 'number' && totalAmount.toFixed) ? totalAmount.toFixed(2) : String(totalAmount)
    }));
    router.push('/signin');
    return;
  }

  // Usuario autenticado: redirigir a la vista de pago /pay con la información en query
  router.push({
    path: '/pay',
    query: {
      play: play.id,
      theater: t.id,
      time: time,
      seats: selectedSeats.value.join(','),
      date: date,
      ticketType: ticketType,
      total: (typeof totalAmount === 'number' && totalAmount.toFixed) ? totalAmount.toFixed(2) : String(totalAmount)
    }
  });
}

// Función de logout
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
/* Estilos principales y de contenido */
.cine-page {
  display: flex;
  /* El height: 100vh se mantiene para que el sidebar pueda usar height: 100vh */
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
  /* Permite que el contenido principal tenga su propio scroll si es necesario */
  overflow-y: auto;
}
.cine-layout {
  display: grid;
  /* CORRECCIÓN FINAL: Permitir a la columna derecha tomar la mitad del espacio disponible */
  grid-template-columns: 1fr minmax(420px, 50%);
  gap: 1.5rem;
  width: 100%;
  max-width: 1200px;
}
.image-frame { display:flex; flex-direction:column; gap:1rem; align-items:center; }
.image-box { width:100%; max-width:720px; height:420px; border-radius:16px; overflow:hidden; background:#f5f5f5; box-shadow:0 8px 24px rgba(0,0,0,0.06); position:relative; display:flex; align-items:center; justify-content:center; }
.image-box img { width:100%; height:100%; object-fit:cover; display:block; }
.img-nav { position:absolute; top:50%; transform:translateY(-50%); background:rgba(0,0,0,0.35); color:#fff; border:none; padding:0.5rem 0.9rem; border-radius:6px; cursor:pointer; font-size:1.6rem; }
.img-nav.left { left:12px; } .img-nav.right { right:12px; }
.image-caption { width:100%; max-width:720px; text-align:left; }
.image-caption h2 { margin:0; color:#223; font-size:1.25rem; }
.image-caption .muted { color:#666; margin:0.25rem 0; }
.image-caption .desc { color:#444; margin-top:0.5rem; }
.theaters-panel { background:transparent; }
.theaters-panel h3 { margin:0 0 0.75rem 0; color:#223; }
.search-wrap { position:relative; margin-bottom:0.75rem; }
.search-wrap input { width:100%; padding:0.5rem 0.6rem; border-radius:6px; border:1px solid #e6e6e6; font-family:'Poppins',sans-serif; }
.suggestions { position:absolute; top:calc(100% + 6px); left:0; right:0; background:#fff; border:1px solid #e6e6e6; border-radius:6px; max-height:180px; overflow:auto; z-index:10; box-shadow:0 8px 18px rgba(0,0,0,0.06); margin:0; padding:0; list-style:none; }
.suggestion-item { padding:0.55rem 0.75rem; cursor:pointer; }
.suggestion-item:hover { background:#f6f6f6; }
.theater-cards { display:flex; flex-direction:column; gap:1rem; }
.theater-card { background:rgba(255,255,255,0.98); border-radius:12px; padding:0.9rem; box-shadow:0 8px 18px rgba(0,0,0,0.06); border:1px solid rgba(0,0,0,0.03); }
.theater-header { display:flex; flex-direction:column; gap:0.125rem; margin-bottom:0.5rem; }
.theater-title { font-weight:600; color:#223; } .theater-sub { font-size:0.85rem; color:#666; }
.price-info { margin:0.5rem 0 0.8rem 0; font-size:1.1rem; color:#c0392b; }
.selectors label { display: block; margin-bottom: 0.5rem; font-weight: 500; color: #444; }
.selectors select, .time-list select { width: 100%; padding: 0.4rem 0.5rem; border-radius: 6px; border: 1px solid #e6e6e6; margin-top: 0.2rem; font-family: 'Poppins', sans-serif; }
.time-list { margin-top:0.8rem; }
.date-picker { margin:0.8rem 0 0.8rem 0; }
.date-picker input[type="date"] { padding:0.4rem 0.5rem; border-radius:6px; border:1px solid #e6e6e6; }
.date-hint { margin:0.35rem 0 0; color:#c0392b; font-size:0.9rem; }
.ticket-count { margin:0.8rem 0; }
.ticket-count input { width:80px; padding:0.3rem; border-radius:6px; border:1px solid #e6e6e6; }

/* ESTILOS DE MAPA DE ASIENTOS Y PANTALLA */
.seat-map-container {
    padding: 0.5rem 0;
    border-top: 1px solid #ddd;
    margin-top: 1rem;
    padding-top: 1rem;
}

.screen {
    background: #000;
    color: #fff;
    text-align: center;
    padding: 0.5rem;
    border-radius: 4px;
    margin-bottom: 1.5rem;
    font-weight: 500;
    font-size: 0.95rem;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    border-bottom: 5px solid #c0392b;
}

.seat-legend { display:flex; gap:0.5rem; align-items:center; margin-bottom:0.5rem; font-size:0.9rem; color:#333; }
.seat { width:34px; height:28px; border-radius:4px; border:1px solid rgba(0,0,0,0.05); margin:2px; background:#e9e9e9; cursor:pointer; font-size:0.8rem; }
.seat.available { background:#f6f6f6; color:#222; }
.seat.selected { background:#556B2F; color:#fff; }
.seat[disabled] { opacity:0.45; cursor:not-allowed; }
.seat-grid { 
    display:flex; 
    flex-direction:column; 
    gap:6px; 
    overflow-x: auto; /* Asegura el scroll horizontal si es necesario */
    padding-right:6px; 
} 
.seat-row { 
    display:flex; 
    align-items:center; 
    gap:6px; 
    width: fit-content; 
} 
.row-label { width:20px; color:#445522; font-weight:600; text-align:center; margin-right:6px; }
.selected-list { margin-top:0.6rem; display:flex; gap:0.4rem; flex-wrap:wrap; align-items:center; }
.sel-item { background:#fff; padding:0.2rem 0.5rem; border-radius:6px; border:1px solid #e6e6e6; font-size:0.9rem; }
.card-actions { 
    display:flex; 
    gap:0.5rem; 
    margin-top:0.6rem; 
    justify-content:flex-end; 
    padding-top: 0.5rem;
}
.btn { padding:0.55rem 0.9rem; border-radius:8px; cursor:pointer; border:1px solid transparent; font-size:0.95rem; }
.btn-primary { background:#556B2F; color:#fff; border-color:#556B2F; }

/* -------------------------------------------------------------------------- */
/* ESTILOS DEL SIDEBAR EXPANDIBLE */
/* -------------------------------------------------------------------------- */

.sidebar {
  background: #556B2F;
  color: #fff;
  width: 70px;
  min-width: 70px;
  transition: width 0.3s;
  padding-top: 40px;
  /* CAMBIO CLAVE: Asegurar que la barra lateral ocupe toda la altura de la ventana y se fije. */
  height: 100vh;
  position: sticky;
  top: 0;
  padding-bottom: 40px;
  box-shadow: 2px 0 8px rgba(85, 107, 47, 0.08);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 2;
  /* Permite el scroll vertical si la lista de enlaces es muy larga */
  overflow-y: auto; 
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


/* -------------------------------------------------------------------------- */
/* Responsive */
/* -------------------------------------------------------------------------- */

@media (max-width: 1100px) {
  .cine-layout { grid-template-columns: 1fr; }
  .theaters-panel { order: 2; margin-top: 1rem; }
  .image-frame { order: 1; }
  .image-box { height: 300px; max-width: 100%; }

  .sidebar { position: relative; }
}

@media (max-width: 480px) {
  .image-box { height: 220px; }
  .card-actions { justify-content: center; }
  .seat { width:26px; height:22px; font-size:0.7rem; }
  .row-label { display:none; }
}
</style>