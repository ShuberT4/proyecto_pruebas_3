
<template>
  <div id="app">
    <header class="py-3 mb-3 border-bottom shadow-sm text-center">
      <div class="container"> 
        <h1 class="mb-2">Mi Biblioteca Digital</h1>
        <div class="auth-controls">
          <button class="btn btn-info btn-sm me-2" v-if="!isAdminLoggedIn" @click="loginAsAdmin">Iniciar Sesión como Admin</button>
          <button class="btn btn-outline-warning btn-sm" v-if="isAdminLoggedIn" @click="logoutAdmin">Cerrar Sesión Admin</button>
          <span v-if="isAdminLoggedIn" class="admin-indicator ms-2">(Modo Admin Activo)</span>
        </div>
      </div>
    </header>

    <div class="container-fluid">
      <div class="row">
        <nav id="sidebarMenu" class="col-md-3 col-lg-2 d-md-block bg-light sidebar border-end">
          <div class="position-sticky pt-3">
            <ul class="nav nav-pills flex-column">
              <li class="nav-item mb-1">
                <a class="nav-link" :class="{ active: currentView === 'dashboard' }" href="#" @click.prevent="changeCurrentView('dashboard')">
                  Panel Administrativo
                </a>
              </li>
              <li class="nav-item mb-1">
                <a class="nav-link" :class="{ active: currentView === 'users' }" href="#" @click.prevent="changeCurrentView('users')">
                  Gestión de Usuarios
                </a>
              </li>
              <li class="nav-item mb-1">
                <a class="nav-link" :class="{ active: currentView === 'catalog' }" href="#" @click.prevent="changeCurrentView('catalog')">
                  Catálogo
                </a>
              </li>
              <li class="nav-item mb-1">
                <a class="nav-link" :class="{ active: currentView === 'loans' }" href="#" @click.prevent="changeCurrentView('loans')">
                  Gestión de Préstamos
                </a>
              </li>
            </ul>
          </div>
        </nav>

        <main class="col-md-9 ms-sm-auto col-lg-10 px-md-4 py-3">
          {/* Aquí van tus componentes que se muestran condicionalmente */}
          <AdminDashboard
            v-if="currentView === 'dashboard' && isAdminLoggedIn"
            @navigate-to="changeCurrentView" 
          />
          <div v-if="currentView === 'dashboard' && !isAdminLoggedIn" class="alert alert-warning unauthorized-message">
            Debes iniciar sesión como administrador para ver este panel.
          </div>

          <UserManagement
            v-if="currentView === 'users'"
            :is-admin-logged-in="isAdminLoggedIn"
          />
          <CatalogView
            v-if="currentView === 'catalog'"
            :is-admin-logged-in="isAdminLoggedIn"
          />
          <LoanManagement
            v-if="currentView === 'loans'"
            :is-admin-logged-in="isAdminLoggedIn"
          />
        </main>
      </div>
    </div>

    <footer class="container-fluid mt-auto py-3 border-top text-center text-muted bg-light">
      <p class="mb-0">&copy; {{ new Date().getFullYear() }} Tu Nombre/Proyecto</p>
    </footer>
  </div>
</template>
<!---->
<script>
// TU SCRIPT DE App.vue SE MANTIENE IGUAL que el que tenías que funcionaba
// con el método changeCurrentView y los demás.
import UserManagement from './components/UserManagement.vue';
import CatalogView from './components/CatalogView.vue';
import LoanManagement from './components/LoanManagement.vue';
import AdminDashboard from './components/AdminDashboard.vue';

export default {
  name: 'App',
  components: { UserManagement, CatalogView, LoanManagement, AdminDashboard },
  data() {
    return {
      currentView: 'catalog', // O tu vista inicial preferida
      isAdminLoggedIn: false
    };
  },
  methods: {
    changeCurrentView(viewName) {
      console.log('Cambiando vista a:', viewName);
      this.currentView = viewName;
    },
    loginAsAdmin() {
      this.isAdminLoggedIn = true;
      localStorage.setItem('biblioteca_isAdminLoggedIn', 'true');
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
html, body, #app {
  height: 100%; /* Para que el footer se pegue abajo si el contenido es corto */
}

#app {
  display: flex;
  flex-direction: column; /* Organiza header, (container-fluid con row), footer verticalmente */
}

.container-fluid.flex-grow-1 { /* Este contenedor crecerá para empujar el footer */
 display: flex; /* Necesario si el contenido dentro (la row) debe llenar altura */
}

/* Ajustes para la barra lateral y el contenido principal si quieres que ocupen toda la altura menos header/footer */
/* .row {
  flex-grow: 1; 
} */

.sidebar {
  /* Ajusta la altura si es necesario, o deja que Bootstrap la maneje. */
  /* Para un fondo oscuro en la barra lateral y texto claro: */
  /* background-color: #343a40 !important; */ /* Color oscuro de Bootstrap */
}

.sidebar .nav-link {
  font-weight: 500;
  /* color: #adb5bd; */ /* Color de texto claro para fondo oscuro */
  color: #0d6efd; /* Color de enlace azul por defecto si usas bg-light */
  padding: 0.75rem 1rem; /* Un poco más de padding */
  border-radius: 0.25rem; /* Bordes redondeados para los pills */
}

.sidebar .nav-link:hover {
  /* background-color: #495057; */ /* Hover más oscuro para fondo oscuro */
  background-color: #e9ecef; /* Hover claro para fondo bg-light */
}

.sidebar .nav-link.active {
  color: #fff;
  background-color: #0d6efd; /* Azul principal de Bootstrap para el activo */
}

/* Contenido principal */
main.col-md-9 {
  /* background-color: #212529; */ /* Si quieres que el área de contenido sea oscura */
  /* color: #f8f9fa; */
  /* min-height: calc(100vh - (altura_header + altura_footer)); */ /* Para asegurar que ocupe espacio */
}


/* Estilos generales que ya tenías pueden ir aquí */
body { 
  margin: 0; 
  background-color: #343a40; /* Fondo oscuro general que parece que tienes/quieres */
  color: #f8f9fa; /* Texto claro general */
}

#app header { 
  background-color: #212529 !important; /* Header más oscuro */
  color: #f8f9fa;
  /* padding: 15px 20px; text-align: center; border-bottom: 1px solid #495057; */
}
#app header h1 { color: #f8f9fa; }


.admin-indicator { font-style: italic; color: #20c997; /* Un verde más brillante para tema oscuro */ }
.unauthorized-message { /* Ajusta si es necesario para tema oscuro */ }

#app footer {
  /* background-color: #212529 !important; */ /* Footer más oscuro */
  /* color: #adb5bd; */
  /* border-top: 1px solid #495057 !important; */
}

/* Si usas bg-light en la sidebar con un tema oscuro general, puede que necesites ajustar sus colores */
.bg-light.sidebar {
  background-color: #2c3034 !important; /* Un gris oscuro para la sidebar */
  border-right: 1px solid #404040 !important;
}
.sidebar .nav-link {
  color: #adb5bd; /* Texto grisáceo para los enlaces de la sidebar oscura */
}
.sidebar .nav-link:hover {
  color: #fff;
  background-color: #495057;
}
.sidebar .nav-link.active {
  color: #fff;
  background-color: #0d6efd; /* Azul sigue estando bien para el activo */
}

</style>

