<template>
  <div class="dashboard">
    <h2>¡Bienvenido, {{ userName }}!</h2>
    <button @click="logout">Cerrar sesión</button>
  </div>
</template>

<script>
export default {
  name: 'DashboardView',
  data() {
    return { userName: '' }
  },
  created() {
    const user = JSON.parse(localStorage.getItem('user'))
    this.userName = user ? user.name : ''
    if (!user) {
      this.$router.replace('/')
    }
  },
  methods: {
    logout() {
      localStorage.removeItem('user')
      this.$router.push('/')
    }
  }
}
</script>

<style scoped>
.dashboard {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: linear-gradient(to right, #1f1f1f, #121212);
  color: white;
}
button {
  margin-top: 1.5rem;
  padding: 0.7rem 1.5rem;
  background: #00bfff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: white;
}
</style>