<template>
  <div class="registro-card">
    <h2>Regístrate</h2>

    <div v-if="errorMessage" class="error-message" role="alert">{{ errorMessage }}</div>
    <div v-if="successMessage" class="success-message">{{ successMessage }}</div>

    <!-- 🧍 USUARIO -->
    <div class="form-group user-group">
      <input
        v-model="form.user"
        type="text"
        placeholder="Nombre de usuario"
        maxlength="64"
        @input="validateUser"
        :class="{ invalid: !validUser && form.user.length > 0 }"
      />

      <!-- Tooltip -->
      <transition name="fade">
        <div v-if="!validUser && form.user.length > 0" class="tooltip">
          El nombre de usuario debe contener solo letras y números (4–10 caracteres).
        </div>
      </transition>
    </div>

    <!-- 🔒 CONTRASEÑA -->
    <div class="form-group password-group">
      <input
        v-model="form.password"
        type="password"
        placeholder="Contraseña"
        maxlength="16"
        @input="validatePass"
        :disabled="!validUser"
        :class="{ invalid: !validPass && form.password.length > 0 }"
      />

      <transition name="fade">
        <div v-if="validUser && !validPass && form.password.length > 0" class="tooltip">
          La contraseña debe tener entre 8 y 16 caracteres sin espacios.
        </div>
      </transition>
    </div>

    <!-- 🔁 CONFIRMAR CONTRASEÑA -->
    <div class="form-group confirm-group">
      <input
        v-model="form.confirm"
        type="password"
        placeholder="Confirmar contraseña"
        maxlength="16"
        :disabled="!validUser || !validPass"
        :class="{ invalid: form.confirm.length > 0 && form.confirm !== form.password }"
      />

      <transition name="fade">
        <div v-if="form.confirm.length > 0 && form.confirm !== form.password" class="tooltip">
          Las contraseñas no coinciden.
        </div>
      </transition>
    </div>

    <!-- 🟢 BOTÓN -->
    <button :disabled="!canCreate || isSending" class="btn-crear" @click="crearCuenta">
      <span v-if="isSending">Creando...</span>
      <span v-else>Crear Cuenta</span>
    </button>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { useRouter } from 'vue-router';

const router = useRouter();

const form = ref({
  user: "",
  password: "",
  confirm: "",
});

const validUser = ref(false);
const validPass = ref(false);

const isSending = ref(false);
const errorMessage = ref('');
const successMessage = ref('');

const API_BASE = import.meta.env.VITE_API_BASE || 'http://localhost:3001';

// ✅ Validar usuario
function validateUser(e) {
  const val = e.target.value;
  // Permitir solo letras y números
  if (!/^[a-zA-Z0-9]*$/.test(val)) {
    form.value.user = val.replace(/[^a-zA-Z0-9]/g, "");
  }

  // Validación: 4 a 10 caracteres
  validUser.value = /^[a-zA-Z0-9]{4,10}$/.test(form.value.user);

  // Si no cumple, limpiar y bloquear campos inferiores
  if (!validUser.value) {
    form.value.password = "";
    form.value.confirm = "";
    validPass.value = false;
  }
}

// ✅ Validar contraseña
function validatePass(e) {
  const val = e.target.value;
  validPass.value = /^\S{8,16}$/.test(val);
}

// ✅ Botón activado solo si todo correcto
const canCreate = computed(() =>
  validUser.value &&
  validPass.value &&
  form.value.password === form.value.confirm
);

// ✅ Acción final (ahora con POST al backend)
async function crearCuenta() {
  errorMessage.value = '';
  successMessage.value = '';

  if (!canCreate.value) {
    errorMessage.value = 'Completa los campos correctamente.';
    return;
  }

  try {
    isSending.value = true;
    const resp = await fetch(`${API_BASE}/api/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ user: form.value.user, password: form.value.password })
    });

    const json = await resp.json().catch(() => ({}));

    if (resp.ok) {
      successMessage.value = 'Cuenta creada correctamente. Redirigiendo al inicio de sesión...';
      // redirigir al login después de 1.2s
      setTimeout(() => router.push('/login'), 1200);
    } else if (resp.status === 409) {
      errorMessage.value = json.message || 'El usuario o correo ya existe.';
    } else if (resp.status === 400) {
      errorMessage.value = json.message || 'Datos no válidos.';
    } else {
      errorMessage.value = json.message || 'Ocurrió un error al crear la cuenta.';
    }
  } catch (err) {
    console.error('[Register] error', err);
    errorMessage.value = 'Error de red. Intenta de nuevo más tarde.';
  } finally {
    isSending.value = false;
  }
}
</script>

<style scoped>
.registro-card {
  width: 320px;
  background: #fdfdf5;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 10px #c8d0b9;
  text-align: center;
  color: #3c4a1e;
  position: relative;
}

.form-group {
  margin-bottom: 14px;
  position: relative;
}

input {
  width: 100%;
  padding: 10px;
  border: 2px solid #7b8b55;
  border-radius: 8px;
  outline: none;
  transition: border 0.3s;
  font-size: 14px;
}

input:disabled {
  background: #f0f0e8;
  cursor: not-allowed;
}

input.invalid {
  border-color: #c0392b;
}

/* Tooltip */
.tooltip {
  position: absolute;
  top: 105%;
  left: 50%;
  transform: translateX(-50%);
  background: #fff4c1;
  color: #4b3f08;
  font-size: 13px;
  padding: 8px 12px;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
  width: 250px;
  text-align: center;
  border: 1px solid #d3c480;
  z-index: 10;
}

/* Mensajes */
.error-message {
  background: #ffecec;
  color: #8b1c1c;
  padding: 8px 10px;
  border-radius: 8px;
  margin-bottom: 8px;
  border: 1px solid #f5c6c6;
}
.success-message {
  background: #e9f7ea;
  color: #24521b;
  padding: 8px 10px;
  border-radius: 8px;
  margin-bottom: 8px;
  border: 1px solid #cfe9d0;
}

/* Animación */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Botón */
.btn-crear {
  width: 100%;
  background: #4e641e;
  color: white;
  border: none;
  padding: 10px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-crear:hover:enabled {
  background: #617d2d;
}

.btn-crear:disabled {
  background: #a3b28f;
  cursor: not-allowed;
}
</style>