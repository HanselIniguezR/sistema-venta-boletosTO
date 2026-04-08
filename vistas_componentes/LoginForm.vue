<template>
  <form @submit.prevent="handleLogin" class="login-form">
    <h2>Iniciar sesión</h2>
    <input v-model="email" type="email" placeholder="Correo electrónico" required />
    <input v-model="password" type="password" placeholder="Contraseña" required />
    <button type="submit">Ingresar</button>
  </form>
</template>

<script>
export default {
  name: 'LoginForm',
  data() {
    return {
      email: '',
      password: ''
    }
  },
  methods: {
    handleLogin() {
      // Aquí va la lógica de autenticación
      console.log('Login con:', this.email, this.password)
    }
  }
}
</script>

<style scoped>
.login-form {
  max-width: 400px;
  margin: auto;
  padding: 2rem;
  background: #1e1e1e;
  border-radius: 8px;
  color: white;
}
input {
  display: block;
  width: 100%;
  margin: 1rem 0;
  padding: 0.5rem;
  border-radius: 4px;
  border: none;
}
button {
  width: 100%;
  padding: 0.7rem;
  background: #007acc;
  color: white;
  border: none;
  border-radius: 4px;
}
</style>