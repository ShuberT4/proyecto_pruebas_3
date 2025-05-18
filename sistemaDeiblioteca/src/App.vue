<template>
<div id="app">
    <header>
      <h1>Mi Biblioteca Digital</h1>
      <div class="auth-controls">
        <button v-if="!isAdminLoggedIn" @click="loginAsAdmin">Iniciar Sesión como Admin</button>
        <button v-if="isAdminLoggedIn" @click="logoutAdmin">Cerrar Sesión Admin</button>
        <span v-if="isAdminLoggedIn" class="admin-indicator">(Modo Admin Activo)</span>
      </div>
      <nav>
        <button @click="currentView = 'dashboard'">Panel Administrativo</button>
        <button @click="currentView = 'users'">Gestión de Usuarios</button>
        <button @click="currentView = 'catalog'; console.log('currentView ahora es:', currentView)">Catálogo</button>
        <button @click="currentView = 'loans'">Gestión de Préstamos</button>
      </nav>
    </header>
    <main>
      <AdminDashboard
        v-if="currentView === 'dashboard' && isAdminLoggedIn"
        @navigate-to="changeCurrentView"
      />
      <div v-if="currentView === 'dashboard' && !isAdminLoggedIn" class="unauthorized-message">
        Debes iniciar sesión como administrador para ver este panel.
      </div>

      <UserManagement
        v-if="currentView === 'users'"
        :is-admin-logged-in="isAdminLoggedIn"
      />
      <CatalogView
      v-if="currentView === 'catalog'"
      :is-admin-logged-in="isAdminLoggedIn"
/>      />
      <LoanManagement
        v-if="currentView === 'loans'"
        :is-admin-logged-in="isAdminLoggedIn"
      />

    </main>
    <footer>
      <p>&copy; {{ new Date().getFullYear() }} Tu Nombre/Proyecto</p>
    </footer>
  </div>
</template>

<script>
import UserManagement from './components/UserManagement.vue';
import CatalogView from './components/CatalogView.vue';
import LoanManagement from './components/LoanManagement.vue';
import AdminDashboard from './components/AdminDashboard.vue';

export default {
  name: 'App',
  components: {
    UserManagement,
    CatalogView,
    LoanManagement,
    AdminDashboard
  },
  data() {
    return {
      currentView: 'catalog', // Vista inicial por defecto (puedes cambiarla)
      isAdminLoggedIn: false
    };
  },
  methods: {
    changeCurrentView(viewName) {
      this.currentView = viewName;
    },
    loginAsAdmin() {
      // Simulación simple sin contraseña:
      this.isAdminLoggedIn = true;
      localStorage.setItem('biblioteca_isAdminLoggedIn', 'true');
      // Opcional: Pedir contraseña con prompt
      // const password = prompt("Ingresa la contraseña de administrador (ej: admin123):");
      // if (password === "admin123") {
      //   this.isAdminLoggedIn = true;
      //   localStorage.setItem('biblioteca_isAdminLoggedIn', 'true');
      // } else if (password !== null) {
      //   alert("Contraseña incorrecta.");
      // }
    },
    logoutAdmin() {
      this.isAdminLoggedIn = false;
      localStorage.setItem('biblioteca_isAdminLoggedIn', 'false');
      if (this.currentView === 'dashboard') {
        this.currentView = 'catalog';
      }
    },
    checkLoginStatus() {
      const adminStatus = localStorage.getItem('biblioteca_isAdminLoggedIn');
      this.isAdminLoggedIn = adminStatus === 'true';
    }
  },
  created() {
    this.checkLoginStatus();
    if (this.currentView === 'dashboard' && !this.isAdminLoggedIn) {
      this.currentView = 'catalog';
    }
  }
};
</script>

<style>
body { margin: 0; font-family: Avenir, Helvetica, Arial, sans-serif; color: #2c3e50; }
#app header { background-color: #f8f9fa; padding: 15px 20px; text-align: center; border-bottom: 1px solid #dee2e6; }
#app header h1 { margin: 0 0 10px 0; }
.auth-controls { margin-bottom: 10px; }
.auth-controls button {
  padding: 6px 10px;
  margin: 0 5px;
  background-color: #17a2b8; /* Color cian para login */
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}
.auth-controls button:hover { background-color: #138496; }
.auth-controls .admin-indicator { margin-left: 10px; font-style: italic; color: #28a745; /* Verde para indicador admin */ }

#app header nav button {
  margin: 0 5px;
  padding: 8px 12px;
  background-color: #6c757d; /* Gris para botones de navegación */
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}
#app header nav button:hover { background-color: #5a6268; }

#app main { padding: 20px; }
#app footer { text-align: center; padding: 10px; margin-top: 30px; font-size: 0.9em; color: #6c757d; border-top: 1px solid #dee2e6; }
.unauthorized-message {
  text-align: center;
  padding: 30px;
  font-size: 1.2em;
  color: #dc3545; /* Rojo para no autorizado */
  border: 1px dashed #dc3545;
  background-color: #f8d7da;
  border-radius: 5px;
}
</style>