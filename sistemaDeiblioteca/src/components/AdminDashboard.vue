<template>
  <div class="admin-dashboard">
    <h2>Panel Administrativo</h2>

    <div class="dashboard-stats">
      <div class="stat-card">
        <h3>Total de Usuarios</h3>
        <p>{{ totalUsers }}</p>
      </div>
      <div class="stat-card">
        <h3>Total de Títulos de Libros</h3>
        <p>{{ totalBookTitles }}</p>
      </div>
      <div class="stat-card">
        <h3>Total de Ejemplares</h3>
        <p>{{ totalBookCopies }}</p>
      </div>
      <div class="stat-card">
        <h3>Préstamos Activos</h3>
        <p>{{ totalActiveLoans }}</p>
      </div>
    </div>

    <hr />

    <div class="dashboard-navigation">
      <h3>Accesos Rápidos</h3>
      <button @click="navigateToView('users')">Gestionar Usuarios</button>
      <button @click="navigateToView('catalog')">Gestionar Catálogo</button>
      <button @click="navigateToView('loans')">Gestionar Préstamos</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AdminDashboard',
  data() {
    return {
      users: [], books: [], loans: [],
      localStorageKeys: {
        users: 'biblioteca_users',
        books: 'biblioteca_books_vue_project',
        loans: 'biblioteca_loans_vue_project'
      }
    };
  },
  computed: {
    totalUsers() { return this.users.length; },
    totalBookTitles() { return this.books.length; },
    totalBookCopies() { return this.books.reduce((sum, book) => sum + (book.totalCopies || 0), 0); },
    totalActiveLoans() { return this.loans.filter(loan => loan.status === 'activo').length; }
  },
  methods: {
    loadDataFromLocalStorage() {
      const storedUsers = localStorage.getItem(this.localStorageKeys.users);
      if (storedUsers) this.users = JSON.parse(storedUsers); else this.users = [];

      const storedBooks = localStorage.getItem(this.localStorageKeys.books);
      if (storedBooks) this.books = JSON.parse(storedBooks); else this.books = [];
      
      const storedLoans = localStorage.getItem(this.localStorageKeys.loans);
      if (storedLoans) this.loans = JSON.parse(storedLoans); else this.loans = [];
    },
    navigateToView(viewName) {
      this.$emit('navigate-to', viewName);
    }
  },
  created() {
    this.loadDataFromLocalStorage();
  },
  // Si el dashboard se mantiene montado y quieres que las estadísticas se actualicen
  // cuando cambian los datos en otros componentes, necesitarías una forma de recargar
  // o escuchar eventos. Por ahora, se actualizan al entrar a la vista del dashboard.
  // O si App.vue fuerza un re-renderizado del dashboard al cambiar datos.
  // Para una actualización más dinámica, se podría usar un 'activated' hook si se usa <keep-alive>
  // o un event bus, o simplemente recargar los datos si la vista se vuelve activa.
  // Por simplicidad, recargaremos al ser creado/montado.
  // Si navegas fuera y vuelves, se volverá a ejecutar created() si el componente se destruye y re-crea.
  // Si el componente se mantiene vivo (ej. con v-show en lugar de v-if, o <keep-alive>),
  // necesitarías el hook 'activated' o un watcher en currentView en App.vue para llamar
  // a un método de actualización en el dashboard.
  // Con v-if, created() se ejecuta cada vez que se muestra.
};
</script>

<style scoped>
.admin-dashboard { margin: 20px; font-family: sans-serif; }
.dashboard-stats { display: flex; flex-wrap: wrap; gap: 20px; margin-bottom: 20px; }
.stat-card { background-color: #f8f9fa; border: 1px solid #dee2e6; border-radius: 5px; padding: 20px; text-align: center; flex-grow: 1; min-width: 200px; }
.stat-card h3 { margin-top: 0; font-size: 1.2em; color: #495057; }
.stat-card p { font-size: 2em; font-weight: bold; color: #007bff; margin-bottom: 0; }
.dashboard-navigation h3 { margin-top: 30px; margin-bottom: 10px; }
.dashboard-navigation button { background-color: #6c757d; color: white; border: none; padding: 10px 20px; margin-right: 10px; margin-bottom: 10px; border-radius: 5px; cursor: pointer; font-size: 1em; }
.dashboard-navigation button:hover { background-color: #5a6268; }
hr { margin-top: 30px; margin-bottom: 30px; }
</style>