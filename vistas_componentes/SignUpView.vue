<template>
  <div class="sign-up-page">
    <div class="monogram-bg"></div>
    <div class="top-bar"></div>
    <img src="/mono.png" alt="Monograma" class="monogram-top-center" />
    <img src="/mono.png" alt="Monograma" class="monogram-single" />
    <form class="sign-up-form" @submit.prevent="handleSubmit">
      <h2 class="form-title">Regístrate</h2>

      <div class="form-group user-group">
        <input
          type="text"
          v-model="form.username"
          required
          placeholder="Nombre de usuario"
          @input="onUserInput"
          @mouseenter="showWarningUser = true"
          @mouseleave="showWarningUser = false"
        />
        <div v-if="showWarningUser" class="user-warning-tooltip">
          El nombre de usuario debe contener únicamente letras y números, y tener entre 4 y 10 caracteres.
        </div>
      </div>

      <div class="form-group">
        <input
          type="email"
          v-model="form.email"
          required
          placeholder="Correo electrónico"
          maxlength="254"
          @mouseenter="showWarningEmail = true"
          @mouseleave="showWarningEmail = false"
          :disabled="isUserEmpty"
          :aria-disabled="isUserEmpty"
          :tabindex="isUserEmpty ? -1 : 0"
        />
        <div v-if="showWarningEmail" class="user-warning-tooltip">
          Ingresa un correo válido (ej. usuario@dominio.com)
        </div>
      </div>

      <div class="form-group password-group">
        <input
          type="password"
          v-model="form.password"
          required
          placeholder="Contraseña"
          maxlength="16"
          :readonly="isUserEmpty || !validUser"
          :disabled="isUserEmpty || !validUser"
          :aria-disabled="isUserEmpty || !validUser"
          :tabindex="(isUserEmpty || !validUser) ? -1 : 0"
          @input="onPassInput"
          @mouseenter="showWarningPass = true"
          @mouseleave="showWarningPass = false"
        />
        <div v-if="showWarningPass" class="password-warning-tooltip">
          La contraseña debe tener entre 8 y 16 caracteres y solo puede tener caracteres especiales al inicio o al final.
        </div>
      </div>

      <div class="form-group">
        <input
          type="password"
          v-model="form.confirmPassword"
          required
          placeholder="Confirmar contraseña"
          maxlength="16"
          :readonly="isUserEmpty || !validPass"
          :disabled="isUserEmpty || !validPass"
          :aria-disabled="isUserEmpty || !validPass"
          :tabindex="(isUserEmpty || !validPass) ? -1 : 0"
          @input="onConfirmInput"
        />
      </div>

  <button type="submit" class="btn-submit" :disabled="!canCreate">Crear Cuenta</button>
    </form>

    <p class="have-account">¿Ya tienes cuenta? Haz clic <a href="#" @click.prevent="goToLogin" class="link-here">aquí</a> para iniciar sesión.</p>
    <div class="bottom-bar"></div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, watch } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

const showWarningUser = ref(false);
const showWarningPass = ref(false);
const showWarningEmail = ref(false);

const validUser = ref(false);
const validPass = ref(false);
const validConfirm = ref(false);

const form = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
});

// Computed helper: true cuando el campo usuario está vacío (sin texto)
const isUserEmpty = computed(() => !form.username || String(form.username).trim() === '');

const userRegex = /^[a-zA-Z0-9]{4,10}$/;
// Simplified password regex to avoid escaping issues in SFC parsing
// Require 8-16 non-space characters
const passRegex = /^\S{8,16}$/;

function sanitizeUsername(value) {
  // remove any non-alphanumeric characters
  return String(value).replace(/[^a-zA-Z0-9]/g, '');
}

function onUserInput(e) {
  // sanitize and update model
  const cleaned = sanitizeUsername(e.target.value || '');
  if (cleaned !== form.username) form.username = cleaned;
  validUser.value = userRegex.test(form.username);
  // if username becomes invalid, clear downstream fields
  if (!validUser.value) {
    form.password = '';
    form.confirmPassword = '';
    validPass.value = false;
    validConfirm.value = false;
  }
}

function onPassInput(e) {
  form.password = e.target.value || '';
  validPass.value = passRegex.test(form.password) && form.password.length >= 8 && form.password.length <= 16;
  if (!validPass.value) {
    form.confirmPassword = '';
    validConfirm.value = false;
  }
}

function onConfirmInput(e) {
  form.confirmPassword = e.target.value || '';
  validConfirm.value = form.confirmPassword === form.password && form.confirmPassword.length > 0;
}

// require a simple email presence check for enabling submit
function validEmailSimple(email) {
  return typeof email === 'string' && email.includes('@') && email.includes('.');
}

const canCreate = computed(() => {
  return validUser.value && validPass.value && validConfirm.value && validEmailSimple(form.email);
});

// keep fields consistent if upper fields change programmatically
watch(() => form.username, (v) => {
  validUser.value = userRegex.test(v || '');
  if (!validUser.value) {
    form.password = '';
    form.confirmPassword = '';
    validPass.value = false;
    validConfirm.value = false;
  }
});

watch(() => form.password, (v) => {
  validPass.value = passRegex.test(v || '') && (v || '').length >= 8 && (v || '').length <= 16;
  if (!validPass.value) {
    form.confirmPassword = '';
    validConfirm.value = false;
  }
});

async function handleSubmit() {
  if (!userRegex.test(form.username)) {
    alert('El nombre de usuario debe contener únicamente letras y números, y tener entre 4 y 10 caracteres.');
    return;
  }
  if (!passRegex.test(form.password) || form.password.length < 8 || form.password.length > 16) {
    alert('La contraseña debe tener entre 8 y 16 caracteres y solo puede tener caracteres especiales al inicio o al final.');
    return;
  }
  if (form.password !== form.confirmPassword) {
    alert('Las contraseñas no coinciden');
    return;
  }
  if (!validEmailSimple(form.email)) {
    alert('Introduce un correo electrónico válido.');
    return;
  }

  try {
    const res = await fetch('http://localhost:3000/api/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        nombre_usuario: form.username,
        correo_electronico: form.email,
        contrasena: form.password
      })
    });
    const data = await res.json();
    if (res.ok) {
      alert(data.message || 'Registro exitoso');
      // Auto-login: intentar autenticar inmediatamente con las mismas credenciales
      try {
        const loginResp = await fetch('http://localhost:3000/api/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ userOrEmail: form.username, password: form.password })
        });

        if (loginResp.ok) {
          const loginData = await loginResp.json();
          localStorage.setItem('isAuthenticated', 'true');
          localStorage.setItem('user', JSON.stringify(loginData.user || {}));
          // Si había una compra pendiente, la reanudamos
          const pending = sessionStorage.getItem('pendingPurchase');
          if (pending) {
            // redirigir a /pay para continuar
            router.push({ path: '/pay', query: JSON.parse(pending) });
            sessionStorage.removeItem('pendingPurchase');
          } else {
            router.push('/home');
          }
          return;
        } else {
          // Si el auto-login falla, redirigir al login para que ingrese manualmente
          router.push('/');
          return;
        }
      } catch (loginErr) {
        console.error('Auto-login falló:', loginErr);
        router.push('/');
        return;
      }
    } else {
      alert(data.error || data.message || 'Error al registrar. Intenta de nuevo.');
    }
  } catch (error) {
    alert('Error al registrar. Intenta de nuevo.');
    console.error(error);
  }
}

function goToLogin() {
  router.push('/');
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;500&display=swap');
.sign-up-page {
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
.top-bar {
  width: 100vw;
  height: 40px;
  background: #556B2F;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
}
.bottom-bar {
  width: 100vw;
  height: 40px;
  background: #556B2F;
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 10;
}
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
.sign-up-form {
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
  font-family: 'Poppins', sans-serif;
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
  font-family: 'Poppins', sans-serif;
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
  font-family: 'Poppins', sans-serif;
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
  font-family: 'Poppins', sans-serif;
  margin-top: 0.5rem;
}
.btn-submit:hover {
  background-color: #445522;
}
</style>