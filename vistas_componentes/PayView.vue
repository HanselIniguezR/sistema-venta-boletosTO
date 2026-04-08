<template>
  <div class="pago-page">
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
      <div class="pago-layout">
        
        <section class="boleto-orden">
          <h2>🎫 Resumen de tu Orden</h2>
          <div class="ticket-container">
            <div class="ticket-header">
              <h3>{{ orderDetails.type === 'CINE' ? 'Función de Cine' : orderDetails.type === 'MUSEO' ? 'Entrada a Museo' : 'Obra de Teatro' }}</h3>
              <h1>{{ orderDetails.title }}</h1>
            </div>
            
            <div class="ticket-details">
              <p><strong>Ubicación/Lugar:</strong> {{ orderDetails.location }}</p>
              <p><strong>Fecha:</strong> {{ formatDate(orderDetails.date) }}</p>
              <p v-if="orderDetails.time"><strong>Hora:</strong> {{ orderDetails.time }}</p>
              <p><strong>Boletos:</strong> {{ orderDetails.count }}</p>
              <p v-if="orderDetails.seats"><strong>Asientos:</strong> {{ orderDetails.seats }}</p>
              <hr />
              <p class="total-price"><strong>Total a Pagar:</strong> ${{ Number(orderDetails.total).toFixed(2) }}</p>
            </div>

            <div class="ticket-qr">
              <p># {{ orderDetails.orderId }}</p>
              <div class="qr-placeholder">CÓDIGO QR SIMULADO</div>
            </div>
          </div>
          <p class="note">Por favor, verifica los detalles antes de proceder al pago.</p>
        </section>

        <aside class="payment-panel">
          <h2>Métodos de Pago</h2>
          
          <div class="payment-group express-pay">
            <h4>Pago Rápido</h4>
            <button class="btn btn-apple-pay" @click="iniciarPago('Apple Pay')">
              <span class="logo-apple" aria-hidden="true"></span> Pagar con Apple Pay
            </button>
            <button class="btn btn-paypal" @click="iniciarPago('PayPal')">
              <span class="logo-paypal" aria-hidden="true"></span> Pagar con PayPal
            </button>
          </div>

          <div class="payment-group card-pay">
            <h4>Tarjeta de Crédito o Débito</h4>
            <form @submit.prevent="iniciarPago('Tarjeta')">
              <div class="form-group">
                <label for="card-number">Número de Tarjeta</label>
                <input
                  type="text"
                  id="card-number"
                  v-model="cardDetails.number"
                  required
                  placeholder="XXXX XXXX XXXX XXXX"
                  maxlength="19"
                  inputmode="numeric"
                  pattern="^[0-9 ]{13,19}$"
                  @input="onCardNumberInput"
                />
              </div>
              <div class="form-group">
                <label for="card-name">Nombre en la Tarjeta</label>
                <input type="text" id="card-name" v-model="cardDetails.name" required />
              </div>
              <div class="card-row">
                <div class="form-group">
                  <label for="card-expiry">Vencimiento</label>
                  <input
                    type="text"
                    id="card-expiry"
                    v-model="cardDetails.expiry"
                    required
                    placeholder="MM/AA"
                    maxlength="5"
                    inputmode="numeric"
                    pattern="^[0-9]{2}\/([0-9]{2})$"
                    @input="onCardExpiryInput"
                  />
                </div>
                <div class="form-group">
                  <label for="card-cvc">CVC</label>
                  <input
                    type="text"
                    id="card-cvc"
                    v-model="cardDetails.cvc"
                    required
                    placeholder="XXX"
                    maxlength="4"
                    inputmode="numeric"
                    pattern="^[0-9]{3,4}$"
                    @input="onCardCvcInput"
                  />
                </div>
              </div>
              <button type="submit" class="btn btn-primary btn-full">
                Pagar ${{ Number(orderDetails.total).toFixed(2) }}
              </button>
            </form>
          </div>
        </aside>

      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

// SIMULACIÓN: Este debe ser el número de teléfono donde se enviará el boleto.
// Debe incluir el código de país (ejemplo: 52 para México).
const userPhoneNumber = '5215512345678'; // Reemplazar con el número real del usuario

// Campos de estado usados en el payload de la compra
const phoneNumber = ref('');
let storedUser = null;
try { storedUser = JSON.parse(localStorage.getItem('user') || 'null'); } catch (e) { storedUser = null; }
const userName = ref(storedUser?.nombre || storedUser?.nombre_usuario || storedUser?.name || storedUser?.usuario || 'Invitado');
const userEmail = ref(storedUser?.correo_electronico || storedUser?.email || storedUser?.correo || storedUser?.correoElectronico || null);

// --- Búsqueda de Título (Simulación de DB) ---
function getItemTitle(id) {
  if (!id) return 'Error de Datos';
  const normalizedId = String(id).toUpperCase();

  const titleMap = {
    'BATMAN': 'The Batman (Cine)',
    'SUPERMAN': 'Superman (Cine)',
    'SPIDERMAN': 'Spider-Man: Brand New Day!',
    'MOD1': 'Grandes Maestros Modernos', // Museo
    'NAT1': 'Era de los Dinosaurios',     // Museo
    'ANT1': 'Culturas Mesoamericanas',   // Museo
    'TEATRO1': 'La Casa de Bernarda Alba',
    'TEATRO2': 'Sueño de una Noche de Verano',
  };

  return titleMap[normalizedId] || String(id).replace(/_/g, ' ');
}

// --- Búsqueda de Ubicación Específica (Simulación de DB) ---
function getItemLocationName(locationId) {
    if (!locationId) return 'Sede Principal';
    const normalizedId = String(locationId).toLowerCase();

    const locationMap = {
        'cinepolis': 'Cinépolis - Plaza Central',
        'cinemex': 'Cinemex - Las Torres',
        'cinemark': 'Cinemark - Gran Explanada',
        'amc': 'AMC Theatres - Metrópolis',
        'museo_arte_moderno': 'Museo Arte Moderno',
        // Puedes añadir más ubicaciones de teatro/museo aquí
    };

    return locationMap[normalizedId] || String(locationId).replace(/_/g, ' ');
}

// --- ESTADO GENERAL Y MENÚ ---
const isExpanded = ref(false);
function expandMenu() { isExpanded.value = true; }
function collapseMenu() { isExpanded.value = false; }
function logout() { 
    try { localStorage.setItem('isAuthenticated', 'false'); } catch (e) {}
    try { sessionStorage.removeItem('pendingPurchase'); } catch (e) {}
    try { sessionStorage.clear(); } catch (e) {}
    try { router.replace('/'); } catch (e) { router.push('/'); }
}
function formatDate(ymd) {
  if (!ymd) return 'N/A';
  const parts = String(ymd).split('-');
  if (parts.length !== 3) return ymd;
  const [y, m, d] = parts;
  return `${d}/${m}/${y}`;
}

// --- DATOS DE LA ORDEN (Recuperados de la URL) ---
const orderDetails = ref({
  type: 'N/A',
  title: 'Cargando...',
  location: 'N/A',
  date: 'N/A',
  time: null,
  count: 0,
  seats: null,
  total: 0.00,
  orderId: String(Math.floor(Math.random() * 1000000)).padStart(6, '0'),
});

onMounted(() => {
  const params = route.params || {};
  const query = route.query || {};

  let orderType = 'N/A';
  const pathSegments = (route.path || '').split('/').filter(Boolean);
  if (pathSegments.includes('cine')) orderType = 'CINE';
  else if (pathSegments.includes('museo')) orderType = 'MUSEO';
  else if (pathSegments.includes('teatro')) orderType = 'TEATRO';

  const itemId = params.id || query.play || query.id || 'N/A';
  const itemTitle = getItemTitle(itemId);
  
  // Determina el ID de la ubicación para buscar su nombre
  const locationId = query.theater || query.museum || 'Sede Principal';
  const locationName = getItemLocationName(locationId);

  const seatsList = query.seats ? String(query.seats) : null;
  const countFromQuery = parseInt(query.count, 10);
  const countResolved = !isNaN(countFromQuery) ? countFromQuery : (seatsList ? seatsList.split(',').length : 1);

  const rawTotal = parseFloat(query.total || 0) || 0;

  orderDetails.value = {
    type: orderType,
    title: itemTitle,
    location: locationName, // USANDO NOMBRE DE UBICACIÓN LEGIBLE
    date: query.date || formatDate(new Date().toISOString().slice(0,10)),
    time: query.time || null,
    count: countResolved,
    seats: seatsList,
    total: Number(rawTotal).toFixed(2),
    orderId: String(Math.floor(Math.random() * 1000000)).padStart(6, '0'),
  };
});

// --- GENERACIÓN DEL MENSAJE DE WHATSAPP ---
function generateWhatsAppMessage() {
    const details = orderDetails.value;
    const nl = '\n'; // Salto de línea

    let seatsInfo = details.seats ? `Asientos: ${details.seats}` : `Boletos: ${details.count}`;

    let message = `¡Tu compra en Ticketing App ha sido exitosa! 🎉${nl}${nl}`;
    message += `🎟️ *DETALLE DEL BOLETO*${nl}`;
    message += `-----------------------${nl}`;
    message += `Evento: ${details.title} (${details.type})${nl}`;
    message += `Lugar: ${details.location}${nl}`;
    message += `Fecha: ${formatDate(details.date)} ${details.time ? 'a las ' + details.time : ''}${nl}`;
    message += `${seatsInfo}${nl}`;
    message += `Total Pagado: $${details.total}${nl}`;
    message += `ID de Orden: #${details.orderId}${nl}${nl}`;
    message += `Guarda este mensaje para acceder a tu evento.`;
    
    // Codifica el mensaje para ser usado en la URL
    return encodeURIComponent(message);
}

// --- PAGO CON TARJETA ---
const cardDetails = ref({
  number: '',
  name: '',
  expiry: '',
  cvc: '',
});

// --- INPUT SANITIZERS: permitir solo dígitos (nombre se mantiene libre) ---
function onCardNumberInput(e) {
  const raw = String(e.target.value || '').replace(/\D/g, '').slice(0, 16);
  const parts = raw.match(/.{1,4}/g) || [];
  cardDetails.value.number = parts.join(' ');
}

function onCardExpiryInput(e) {
  const raw = String(e.target.value || '').replace(/\D/g, '').slice(0, 4);
  if (raw.length <= 2) {
    cardDetails.value.expiry = raw;
  } else {
    cardDetails.value.expiry = raw.slice(0,2) + '/' + raw.slice(2);
  }
}

function onCardCvcInput(e) {
  const raw = String(e.target.value || '').replace(/\D/g, '').slice(0, 4);
  cardDetails.value.cvc = raw;
}

// --- FUNCIÓN DE PAGO (Redirecciona a WhatsApp) ---
async function iniciarPago(metodo) {
  const total = orderDetails.value.total;

  if (metodo === 'Tarjeta') {
    if (!cardDetails.value.number || !cardDetails.value.name || !cardDetails.value.expiry || !cardDetails.value.cvc) {
      alert('Por favor, completa todos los detalles de la tarjeta.');
      return;
    }
  }

  // 1. Muestra mensaje de éxito de la compra
  alert('¡Compra exitosa! 🎉 Su pago con ' + metodo + ' por $' + Number(total).toFixed(2) + ' ha sido procesado.');
  
  // 2. Intentar envío automático vía backend (/api/boletos). Si falla el intento (no configurado),
  //    caemos al comportamiento antiguo (abrir WhatsApp) como fallback.
  const phoneTo = phoneNumber.value && phoneNumber.value.trim() ? phoneNumber.value.trim() : userPhoneNumber;
    const payload = {
    nombre: userName.value || 'Invitado',
    correo: userEmail.value || null,
    telefono: phoneTo,
    tipo_evento: orderDetails.value.type,
    evento_title: orderDetails.value.title,
    ubicacion: orderDetails.value.location,
    fecha: orderDetails.value.date,
    hora: orderDetails.value.time,
    asientos: orderDetails.value.seats,
    cantidad: orderDetails.value.count,
      medio_pago: metodo,
    total: Number(orderDetails.value.total) || total,
    order_code: orderDetails.value.orderId,
    user_id: (storedUser && storedUser.id) ? storedUser.id : null
  };

  try {
    const envBase = (typeof import.meta !== 'undefined' && import.meta.env && import.meta.env.VITE_API_BASE) ? import.meta.env.VITE_API_BASE : null;
    const guessedBase = (!envBase && location && location.hostname === 'localhost') ? 'http://localhost:3001' : '';
    const base = (envBase && envBase.trim() !== '') ? envBase.replace(/\/$/, '') : (guessedBase ? guessedBase : '');
    const url = base ? `${base.replace(/\/$/, '')}/api/boletos` : '/api/boletos';

    const resp = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    let data = null;
    try { data = await resp.json(); } catch (e) { data = null; }

    if (!resp.ok) {
      // If backend rejected or errored, fall back to opening WhatsApp with prefilled message
      console.warn('[PayView] backend /api/boletos error, falling back to client WhatsApp. status=', resp.status, 'body=', data);
      const message = generateWhatsAppMessage();
      const whatsappLink = `https://wa.me/${phoneTo}?text=${message}`;
      window.open(whatsappLink, '_blank');
      setTimeout(() => router.push('/home'), 1500);
      return;
    }

    // If backend returned a PDF URL, open it
    const pdfUrl = data && (data.pdf || data.url) ? (data.pdf || data.url) : null;
    if (pdfUrl) {
      const absolute = pdfUrl.startsWith('http') ? pdfUrl : (location.origin.replace(/:\d+$/, '') + pdfUrl);
      window.open(absolute, '_blank');
    }

    const tw = data && data.twilio ? data.twilio : null;
    if (tw && tw.attempted && tw.success) {
      alert('Boleto creado y enviado por WhatsApp correctamente.');
    } else if (tw && tw.attempted && !tw.success) {
      const reason = tw.reason || 'Error desconocido';
      alert('Boleto creado. Falló el envío por WhatsApp: ' + reason + '\nPuedes reenviar desde Mis Boletos.');
    } else if (tw && !tw.attempted) {
      // Backend didn't try (missing config/phone) -> fallback: open WhatsApp so user can send manually
      const reason = tw.reason || 'no attempt recorded';
      console.info('[PayView] backend did not attempt Twilio send:', reason);
      const message = generateWhatsAppMessage();
      const whatsappLink = `https://wa.me/${phoneTo}?text=${message}`;
      window.open(whatsappLink, '_blank');
      alert('Boleto creado. No se pudo enviar automáticamente por WhatsApp: ' + reason + '\nSe abrió WhatsApp para que lo envíes manualmente.');
    } else {
      // Unknown/missing tw info: just show created
      alert('Boleto creado. ' + (pdfUrl ? 'PDF disponible.' : 'Revise su correo o la sección de boletos.'));
    }

    // finalize and redirect
    try { sessionStorage.removeItem('pendingPurchase'); } catch(e){}
    setTimeout(() => router.push('/home'), 1200);

  } catch (err) {
    console.error('[PayView] error calling /api/boletos:', err);
    // fallback to opening WhatsApp if backend call fails
    const message = generateWhatsAppMessage();
    const whatsappLink = `https://wa.me/${phoneTo}?text=${message}`;
    window.open(whatsappLink, '_blank');
    setTimeout(() => router.push('/home'), 1500);
  }
}
</script>

<style scoped>
/* -------------------------------------------------------------------------- */
/* ESTILOS BASE Y BARRA LATERAL */
/* -------------------------------------------------------------------------- */
.pago-page {
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
/* ESTILOS DE PAGO Y BOLETO */
/* -------------------------------------------------------------------------- */

.pago-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  width: 100%;
  max-width: 1000px;
  margin-top: 2rem;
}

/* SECCIÓN IZQUIERDA: BOLETO */
.boleto-orden {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.boleto-orden h2 {
    color: #556B2F;
    font-weight: 600;
}
.ticket-container {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  border-left: 8px solid #556B2F;
}
.ticket-header h1 {
  font-size: 1.8rem;
  color: #223;
  margin-bottom: 0.5rem;
}
.ticket-header h3 {
  font-size: 1rem;
  color: #556B2F;
  margin-top: 0;
}
.ticket-details p {
  margin: 0.5rem 0;
  font-size: 1rem;
  color: #444;
}
.ticket-details hr {
    border: none;
    border-top: 1px dashed #ccc;
    margin: 1rem 0;
}
.total-price {
  font-size: 1.4rem !important;
  color: #c0392b !important;
  font-weight: 700;
}
.ticket-qr {
    text-align: center;
    margin-top: 1.5rem;
    padding-top: 1rem;
    border-top: 1px solid #eee;
}
.ticket-qr p {
    font-size: 0.9rem;
    color: #777;
    margin-bottom: 0.5rem;
}
.qr-placeholder {
    width: 100px;
    height: 100px;
    background: #e0e0e0;
    margin: 10px auto 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.75rem;
    color: #666;
    border-radius: 4px;
}
.note {
    font-size: 0.9rem;
    color: #777;
}

/* SECCIÓN DERECHA: PAGOS */
.payment-panel h2 {
    color: #556B2F;
    font-weight: 600;
    margin-bottom: 1.5rem;
}
.payment-group {
    background: #f4f4e7;
    border-radius: 12px;
    padding: 1.2rem;
    margin-bottom: 1.5rem;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}
.payment-group h4 {
    margin-top: 0;
    margin-bottom: 1rem;
    color: #556B2F;
}
.btn {
    padding: 0.75rem 1rem;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 600;
    font-size: 1rem;
    width: 100%;
    margin-bottom: 0.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
}
/* LOGOS ACTUALIZADOS */
.logo-apple {
    font-size: 1.3rem; 
    font-weight: 700;
    margin-right: 0.5rem;
    /* Fuente que suele soportar el ícono de Apple */
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
}
.logo-paypal {
    width: 24px;
    height: 18px;
    background: url('https://upload.wikimedia.org/wikipedia/commons/b/b5/PayPal.svg') no-repeat center;
    background-size: contain;
    margin-right: 0.5rem;
}

.btn-apple-pay {
    background: #000;
    color: #fff;
}
.btn-paypal {
    background: #ffc439;
    color: #163659;
}
.btn-primary {
    background: #556B2F;
    color: #fff;
}

/* Formulario de Tarjeta */
.card-pay form {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
}
.form-group label {
    display: block;
    font-size: 0.9rem;
    color: #444;
    margin-bottom: 0.2rem;
}
.form-group input {
    width: 100%;
    padding: 0.6rem;
    border-radius: 4px;
    border: 1px solid #ccc;
    font-family: 'Poppins', sans-serif;
    box-sizing: border-box;
}
.card-row {
    display: flex;
    gap: 1rem;
}
.card-row .form-group {
    flex: 1;
}
.btn-full {
    margin-top: 1rem;
}

/* Responsive */
@media (max-width: 850px) {
    .pago-layout {
        grid-template-columns: 1fr;
        max-width: 600px;
    }
}
</style>