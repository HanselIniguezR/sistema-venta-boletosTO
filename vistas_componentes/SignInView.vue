<template>
  <div class="sign-in-page">
    <div class="monogram-bg" :style="{ backgroundImage: `url(${monogramUrl})` }"></div>
    <div class="top-bar"></div>
    <img :src="monoUrl" alt="Monograma" class="monogram-top-center" />
    <img :src="monoUrl" alt="Monograma" class="monogram-single" />

    <form class="sign-in-form" @submit.prevent="handleSignIn">
      <h2 class="form-title">Iniciar sesión</h2>
      <!-- Depuración: estado reactivo visible -->
      <div class="debug-state" style="font-size:0.85rem;color:#666;margin-bottom:0.6rem;">
        Estado (debug): usuario válido: {{ validUser }} · contraseña válida: {{ validPass }}
      </div>
      <div v-if="errorMessage" class="error-message" role="alert">{{ errorMessage }}</div>

      <!-- Campo Usuario -->
      <div class="form-group user-group">
        <div class="input-with-action">
          <input
            ref="userRef"
            type="text"
            v-model="form.userOrEmail"
            required
            placeholder="Correo o nombre de usuario"
            maxlength="64"
            pattern="[a-zA-Z0-9@._+\- ]{1,64}"
            @input="onUserInput"
            @mouseenter="showWarningUser = true"
            @mouseleave="showWarningUser = false"
          />
        </div>
        <div v-if="showWarningUser || errorMessage" class="user-warning-tooltip">
          {{ errorMessage || userTooltip }}
        </div>
      </div>

      <!-- Campo Contraseña -->
      <div class="form-group password-group">
        <div class="input-with-action">
          <input
            ref="passRef"
            type="password"
            v-model="form.password"
            required
            placeholder="Contraseña"
            @input="onPassInput"
            @mouseenter="showWarningPass = true"
            @mouseleave="showWarningPass = false"
            :readonly="!validUser"
            :disabled="!validUser"
            :aria-disabled="!validUser"
            :tabindex="validUser ? 0 : -1"
          />
        </div>
        <div v-if="showWarningPass" class="password-warning-tooltip">
          La contraseña debe tener entre 8 y 16 caracteres y solo puede tener caracteres especiales al inicio o al final.
        </div>
      </div>

      <!-- Botones -->
      <button
        type="submit"
        class="btn-submit"
        :disabled="!validUser || !validPass"
        :aria-disabled="!validUser || !validPass"
      >
        Iniciar sesión
      </button>
      <button type="button" class="btn-link" @click="showReset = true">Recuperar contraseña</button>
    </form>

    <!-- Modal de recuperación -->
    <div v-if="showReset" class="reset-modal-overlay" @click.self="closeReset">
      <div class="reset-modal">
        <h3>Recuperar contraseña</h3>
        <p>Introduce tu correo o nombre de usuario y te enviaremos instrucciones para restablecer la contraseña.</p>
        <input type="text" v-model="resetEmail" placeholder="Correo o nombre de usuario" class="reset-input" />
        <div class="reset-actions">
          <button class="btn-submit" @click="handleSendReset" :disabled="isSending">Enviar enlace</button>
          <button class="btn-cancel" @click="closeReset">Cancelar</button>
        </div>
        <div v-if="resetMessage" class="reset-message">{{ resetMessage }}</div>
      </div>
    </div>

    <div class="bottom-bar"></div>
  </div>
</template>

<script setup>
import { reactive, ref, watch, computed } from 'vue'; // **IMPORTAR 'watch' y 'computed'**
import { useRouter } from 'vue-router';
import mono from '../assets/mono.png';
import monogram from '../assets/monogram.png';
import { resumePending } from '../composables/usePurchase';

const router = useRouter();

// Estado UI
const showWarningUser = ref(false);
const showWarningPass = ref(false);
const errorMessage = ref('');
// const isUserInputAllowed = ref(true); // Ya no es necesario

// Refs DOM
const userRef = ref(null);
const passRef = ref(null);

// Validaciones
const validUser = ref(false);
const validPass = ref(false);

// Recurso extra para el requisito
const userHasTyped = ref(false); // Para mostrar el mensaje solo si el usuario ha escrito.

// Recursos
const monoUrl = mono;
const monogramUrl = monogram;

const form = reactive({
  userOrEmail: '',
  password: ''
});

const showReset = ref(false);
const resetEmail = ref('');
const resetMessage = ref('');
const isSending = ref(false);

const API_BASE = import.meta.env.VITE_API_BASE || 'http://localhost:3001';

// Función auxiliar para verificar el formato de usuario/correo
const userRegex = /^[a-zA-Z0-9]{4,10}$|^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const passRegex = /^\S{8,16}$/;
const allowedChars = /^[a-zA-Z0-9@._+-]*$/; // Caracteres permitidos

// Mensaje dinámico para tooltip según si el usuario ingresó algo que parece correo
const userTooltip = computed(() => {
  const val = form.userOrEmail || '';
  if (val.includes('@')) {
    return 'Ingresa un correo válido (ej. usuario@dominio.com)';
  }
  return 'El nombre de usuario debe contener únicamente letras y números, y tener entre 4 y 10 caracteres.';
});

// **NUEVA FUNCIÓN DE CÁLCULO DE MENSAJE DE ERROR**
function updateErrorMessage() {
  if (!userHasTyped.value) {
    errorMessage.value = ''; // No mostrar error si no se ha escrito nada.
    return;
  }
  
  const val = form.userOrEmail;
  
  if (!allowedChars.test(val)) {
    errorMessage.value = 'Caracteres no permitidos en usuario/correo.';
    validUser.value = false;
    return;
  }

  // **Lógica del requisito: Mostrar mensaje si el formato NO es válido**
  if (!userRegex.test(val)) {
    // Si el usuario escribió algo con '@' mostrar mensaje de correo
    if (val.includes('@')) {
      errorMessage.value = 'Ingresa un correo válido (ej. usuario@dominio.com)';
    } else {
      errorMessage.value = 'Introduce un nombre de usuario válido (4-10 letras/números) o un correo electrónico válido.';
    }
    validUser.value = false;
    // La contraseña y el botón de submit se bloquean si validUser es false
  } else {
    // Si el usuario es válido, se quita el mensaje de error del usuario
    errorMessage.value = '';
    validUser.value = true;
  }
}

// **AJUSTE EN onUserInput**
function onUserInput(e) {
  // Aquí usamos 'e' solo para la entrada en tiempo real, pero el valor final está en form.userOrEmail
  userHasTyped.value = true;
  
  const val = (e && e.target ? e.target.value : form.userOrEmail) || '';
  
  // 1. Verificar caracteres (Si no pasan, validUser es false y sale el mensaje de error).
  if (!allowedChars.test(val)) {
     validUser.value = false;
     updateErrorMessage();
     return;
  }

  // 2. Aplicar validación de formato y actualizar el error.
  validUser.value = userRegex.test(val);
  updateErrorMessage();
  console.log('[SignIn] onUserInput', { val, validUser: validUser.value, errorMessage: errorMessage.value });
  
  // Intentar enfocar la contraseña solo si el usuario es válido
  if (validUser.value && passRef.value) {
    passRef.value.focus();
  }
}

// **AJUSTE EN onPassInput (Simplificado)**
function onPassInput(e) {
  const val = (e && e.target ? e.target.value : form.password) || '';
  validPass.value = passRegex.test(val);
  
  // Limpiar el error de contraseña si la contraseña es válida
  if (validPass.value && errorMessage.value.startsWith('La contraseña')) {
    errorMessage.value = '';
  } else if (!validPass.value && validUser.value) {
     // Opcional: Mostrar error de contraseña solo si el usuario ya es válido
     // errorMessage.value = 'La contraseña debe tener entre 8 y 16 caracteres sin espacios.';
  }
  console.log('[SignIn] onPassInput', { val, validPass: validPass.value });
}

// **WATCH para manejar el mensaje de error de la contraseña si el usuario se vuelve inválido**
// Si el usuario se vuelve inválido, debe sobrescribir cualquier mensaje de contraseña.
watch(validUser, (newVal) => {
    if (!newVal) {
        // Al volverse inválido, actualizar el mensaje de error del usuario.
        updateErrorMessage();
        // También resetear la contraseña y su validez para que no quede la validación
        // de contraseña "guardada" al desbloquear.
        form.password = '';
        validPass.value = false;
    }
});

// **AJUSTE EN handleSignIn (Eliminar validaciones redundantes que ya hace la interfaz)**
async function handleSignIn() {
  try {
    errorMessage.value = '';
    
    // Las validaciones de la interfaz ya controlan el estado de los campos
    if (!validUser.value || !validPass.value) {
        // En teoría, este bloque no debería ejecutarse si el botón está deshabilitado
        // y el mensaje de error está visible, pero es una protección.
        updateErrorMessage();
        onPassInput(); // para actualizar el error de contraseña si es necesario
        return;
    }
    
    // ... (El resto de la función handleSignIn sigue igual)
    const resp = await fetch(`${API_BASE}/api/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ userOrEmail: form.userOrEmail, password: form.password })
    });

    const json = await resp.json().catch(() => ({}));
    if (resp.ok) {
      localStorage.setItem('isAuthenticated', 'true');
      localStorage.setItem('user', JSON.stringify(json.user || {}));
      if (resumePending(router)) return;
      router.push('/home');
    } else {
      errorMessage.value = 'Usuario o contraseña inválidos';
    }
  } catch (error) {
    errorMessage.value = 'Error al iniciar sesión. Intenta de nuevo.';
    console.error('[SignIn] exception:', error);
  }
}

function closeReset() {
  showReset.value = false;
  resetEmail.value = '';
  resetMessage.value = '';
  isSending.value = false;
}

async function handleSendReset() {
  try {
    resetMessage.value = '';
    if (!resetEmail.value || resetEmail.value.trim().length < 3) {
      resetMessage.value = 'Introduce un correo o usuario válido.';
      return;
    }
    isSending.value = true;
    const resp = await fetch(`${API_BASE}/api/request-password-reset`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ userOrEmail: resetEmail.value.trim() })
    });
    if (resp.ok) {
      resetMessage.value = 'Si existe una cuenta con ese dato, se han enviado instrucciones al correo asociado.';
    } else {
      resetMessage.value = 'Si existe una cuenta con ese dato, se han enviado instrucciones al correo asociado.';
    }
  } catch (err) {
    console.error('[Reset] error:', err);
    resetMessage.value = 'Ocurrió un error al solicitar la recuperación. Intenta de nuevo más tarde.';
  } finally {
    isSending.value = false;
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;500&display=swap');
.sign-in-page {
  height: 100vh;
  background-color: #FFFDD0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
  padding: 0;
  font-family: 'Poppins', sans-serif;
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
.top-bar, .bottom-bar {
  width: 100vw;
  height: 40px;
  background: #556B2F;
  position: fixed;
  left: 0;
  z-index: 10;
}
.top-bar { top: 0; }
.bottom-bar { bottom: 0; }
.monogram-single {
  position: fixed;
  left: 0;
  bottom: 40px;
  width: 80px;
  height: 80px;
  object-fit: contain;
  z-index: 15;
}
.monogram-top-center {
  position: fixed;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 80px;
  object-fit: contain;
  z-index: 15;
  margin-top: -20px;
}
.input-with-action {
  display: flex;
  align-items: center;
}
.input-with-action input[readonly] {
  background: #f5f5f5;
  color: #666;
  cursor: not-allowed;
}
/* Estilo equivalente para inputs deshabilitados */
.input-with-action input[disabled] {
  background: #f5f5f5;
  color: #666;
  cursor: not-allowed;
}
.sign-in-form {
  background-color: #ffffff;
  padding: 1.2rem 1.2rem;
  border-radius: 30px;
  box-shadow: 0 4px 18px rgba(85, 107, 47, 0.12);
  width: 270px;
  min-height: 270px;
  z-index: 20;
  text-align: center;
  font-family: 'Poppins', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 60px;
  margin-bottom: 60px;
  justify-content: center;
  position: relative;
}
.form-title {
  font-size: 1.2rem;
  font-weight: 500;
  margin-bottom: 1.2rem;
  color: #556B2F;
}
.user-group, .password-group {
  position: relative;
  width: 100%;
}
.user-warning-tooltip,
.password-warning-tooltip {
  position: absolute;
  top: 110%;
  left: 50%;
  transform: translateX(-50%);
  background: #fffbe6;
  color: #b8860b;
  border: 1px solid #ffe58f;
  border-radius: 8px;
  padding: 0.5rem 0.7rem;
  font-size: 0.95rem;
  text-align: center;
  white-space: pre-line;
  box-shadow: 0 2px 8px rgba(85, 107, 47, 0.12);
  z-index: 100;
  min-width: 220px;
}
.form-group {
  width: 100%;
  margin-bottom: 0.7rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.form-group input {
  width: 100%;
  padding: 0.5rem 0.7rem;
  border: 1.5px solid #556B2F;
  border-radius: 6px;
  font-size: 0.95rem;
  color: #556B2F;
  background: #f8f8f8;
  outline: none;
  text-align: center;
  transition: border-color 0.2s;
}
.form-group input:focus {
  border-color: #445522;
  background: #fff;
}
.btn-submit {
  width: 100%;
  padding: 0.7rem;
  background-color: #556B2F;
  color: #ffffff;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 0.5rem;
}
.btn-submit:hover { background-color: #445522; }
.btn-link {
  margin-top: 0.5rem;
  background: transparent;
  border: none;
  color: #445522;
  cursor: pointer;
  text-decoration: underline;
}
.reset-modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 60;
}
.reset-modal {
  background: #fff;
  padding: 1.25rem;
  border-radius: 12px;
  width: 320px;
  box-shadow: 0 8px 30px rgba(0,0,0,0.2);
  text-align: center;
}
.reset-input {
  width: 100%;
  padding: 0.5rem;
  margin-top: 0.75rem;
  margin-bottom: 0.5rem;
  border: 1px solid #dcdcdc;
  border-radius: 6px;
}
.reset-actions { display:flex; gap:0.5rem; justify-content:center; margin-top:0.5rem; }
.btn-cancel { background:#eee; color:#333; border:none; padding:0.5rem 0.9rem; border-radius:6px; cursor:pointer; }
.reset-message { margin-top:0.6rem; color:#333; font-size:0.95rem; }
</style>