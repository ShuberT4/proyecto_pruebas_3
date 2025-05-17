<template>
  <div class="loan-management">
    <h2>Gestión de Préstamos</h2>

    <div class="loan-form-section" v-if="isAdminLoggedIn"> {/* O podrías quitar v-if si cualquier usuario puede registrar */}
      <h3>Registrar Nuevo Préstamo</h3>
      <form @submit.prevent="handleRegisterLoan">
        <div>
          <label for="loan-user">Usuario:</label>
          <select id="loan-user" v-model="newLoan.userId" required>
            <option disabled value="">Seleccione un usuario</option>
            <option v-for="user in availableUsersForLoan" :key="user.id" :value="user.id">
              {{ user.name }} ({{ user.email }})
            </option>
          </select>
        </div>
        <div>
          <label for="loan-book">Libro:</label>
          <select id="loan-book" v-model="newLoan.bookId" required>
            <option disabled value="">Seleccione un libro</option>
            <option v-for="book in availableBooksForLoan" :key="book.id" :value="book.id">
              {{ book.title }} (Disponibles: {{ book.availableCopies }})
            </option>
          </select>
        </div>
        <div>
          <label for="loan-dueDate">Fecha de Devolución Esperada:</label>
          <input type="date" id="loan-dueDate" v-model="newLoan.dueDate" required />
        </div>
        <button type="submit">Registrar Préstamo</button>
      </form>
    </div>
     <div v-else class="admin-only-message">
        <p><em>El registro de préstamos está restringido a administradores.</em></p>
        <p><em>(Si deseas que los usuarios puedan registrar préstamos, esta lógica debe ajustarse)</em></p>
    </div>


    <hr />

    <div class="active-loans-section">
      <h3>Préstamos Activos</h3>
      <div v-if="!isAdminLoggedIn && activeLoansForCurrentUser.length > 0">
          <p><em>Mostrando solo tus préstamos activos. Los administradores pueden ver todos.</em></p>
      </div>
      <table v-if="loansToDisplay.active.length > 0">
        <thead>
          <tr>
            <th>Libro</th>
            <th>Usuario</th>
            <th>Fecha Préstamo</th>
            <th>Fecha Devolución</th>
            <th v-if="isAdminLoggedIn">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="loan in loansToDisplay.active" :key="loan.id">
            <td>{{ getBookTitle(loan.bookId) }}</td>
            <td>{{ getUserName(loan.userId) }}</td>
            <td>{{ formatDate(loan.loanDate) }}</td>
            <td>{{ formatDate(loan.dueDate) }}</td>
            <td v-if="isAdminLoggedIn">
              <button @click="handleReturnBook(loan.id)">Marcar como Devuelto</button>
            </td>
          </tr>
        </tbody>
      </table>
      <p v-else>No hay préstamos activos {{ isAdminLoggedIn ? '' : 'para tu usuario' }}.</p>
    </div>

    <hr />

    <div class="returned-loans-section">
      <h3>Historial de Préstamos (Devueltos)</h3>
       <div v-if="!isAdminLoggedIn && returnedLoansForCurrentUser.length > 0">
          <p><em>Mostrando solo tu historial de préstamos. Los administradores pueden ver todos.</em></p>
      </div>
      <table v-if="loansToDisplay.returned.length > 0">
        <thead>
          <tr>
            <th>Libro</th>
            <th>Usuario</th>
            <th>Fecha Préstamo</th>
            <th>Fecha Devolución Esperada</th>
            <th>Fecha Devolución Real</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="loan in loansToDisplay.returned" :key="loan.id">
            <td>{{ getBookTitle(loan.bookId) }}</td>
            <td>{{ getUserName(loan.userId) }}</td>
            <td>{{ formatDate(loan.loanDate) }}</td>
            <td>{{ formatDate(loan.dueDate) }}</td>
            <td>{{ formatDate(loan.returnDate) }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>No hay préstamos en el historial {{ isAdminLoggedIn ? '' : 'para tu usuario' }}.</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoanManagement',
  props: { // Recibir la prop
    isAdminLoggedIn: {
      type: Boolean,
      default: false
    }
    // Podríamos pasar el ID del usuario "logueado" si no es admin para filtrar sus préstamos
    // currentUserId: { type: String, default: null } // Ejemplo
  },
  data() {
    return {
      loans: [], users: [], books: [],
      newLoan: { userId: '', bookId: '', dueDate: this.getDefaultDueDate() },
      nextLoanIdCounter: 1,
      localStorageKeys: {
        users: 'biblioteca_users',
        books: 'biblioteca_books_vue_project',
        loans: 'biblioteca_loans_vue_project'
      }
    };
  },
  computed: {
    availableUsersForLoan() {
      // Solo usuarios no bloqueados pueden tomar préstamos
      return this.users.filter(user => !user.isBlocked);
    },
    availableBooksForLoan() {
      return this.books.filter(book => book.availableCopies > 0);
    },
    allActiveLoans() {
      return this.loans.filter(loan => loan.status === 'activo');
    },
    allReturnedLoans() {
      return this.loans.filter(loan => loan.status === 'devuelto');
    },
    // --- Lógica para mostrar préstamos si el usuario no es admin (necesitaría currentUserId) ---
    // Esto es un ejemplo, necesitaríamos saber el ID del usuario "normal" logueado.
    // Por ahora, asumimos que si no es admin, no ve la lista o ve una vacía hasta implementar login de usuario normal.
    activeLoansForCurrentUser() {
      // if (this.currentUserId && !this.isAdminLoggedIn) {
      //  return this.allActiveLoans.filter(loan => loan.userId === this.currentUserId);
      // }
      return []; // Por defecto, si no hay un usuario normal logueado identificado
    },
    returnedLoansForCurrentUser() {
      // if (this.currentUserId && !this.isAdminLoggedIn) {
      //  return this.allReturnedLoans.filter(loan => loan.userId === this.currentUserId);
      // }
      return [];
    },
    loansToDisplay() {
      if (this.isAdminLoggedIn) {
        return { active: this.allActiveLoans, returned: this.allReturnedLoans };
      }
      // Si no es admin, necesitaríamos un `currentUserId` para mostrar solo sus préstamos.
      // Por ahora, para simplificar y dado que no tenemos login de usuario normal,
      // mostraremos listas vacías o un mensaje. El formulario de registro se oculta.
      return { active: this.activeLoansForCurrentUser, returned: this.returnedLoansForCurrentUser };
    }
  },
  methods: {
    getDefaultDueDate() { /* ...tu código... */ const today = new Date(); today.setDate(today.getDate() + 7); return today.toISOString().split('T')[0]; },
    formatDate(dateString) { /* ...tu código... */ if (!dateString) return 'N/A'; const date = new Date(dateString); return date.toLocaleDateString('es-CO', { year: 'numeric', month: 'long', day: 'numeric' }); },
    loadDataFromLocalStorage() {
      const storedUsers = localStorage.getItem(this.localStorageKeys.users);
      if (storedUsers) this.users = JSON.parse(storedUsers).map(u => ({...u, isBlocked: u.isBlocked || false})); else this.users = [];

      const storedBooks = localStorage.getItem(this.localStorageKeys.books);
      if (storedBooks) this.books = JSON.parse(storedBooks); else this.books = [];
      
      const storedLoans = localStorage.getItem(this.localStorageKeys.loans);
      if (storedLoans) {
        try {
          this.loans = JSON.parse(storedLoans);
          if (this.loans.length > 0) {
            const maxIdNum = this.loans.reduce((max, loan) => {
                const idNum = parseInt(loan.id.substring(2)); // 'ln' tiene 2 caracteres
                return !isNaN(idNum) && idNum > max ? idNum : max;
            }, 0);
            this.nextLoanIdCounter = maxIdNum + 1;
          } else { this.nextLoanIdCounter = 1; }
        } catch (e) { console.error('Error parseando préstamos:', e); this.loans = []; this.nextLoanIdCounter = 1; }
      } else { this.loans = []; this.nextLoanIdCounter = 1; }
    },
    saveDataToLocalStorage() { /* ...tu código... */ localStorage.setItem(this.localStorageKeys.books, JSON.stringify(this.books)); localStorage.setItem(this.localStorageKeys.loans, JSON.stringify(this.loans)); },
    generateLoanId() { return 'ln' + this.nextLoanIdCounter++; },
    getBookTitle(bookId) { /* ...tu código... */ const book = this.books.find(b => b.id === bookId); return book ? book.title : 'Libro Desconocido'; },
    getUserName(userId) { /* ...tu código... */ const user = this.users.find(u => u.id === userId); return user ? user.name : 'Usuario Desconocido'; },
    handleRegisterLoan() {
      if (!this.isAdminLoggedIn) { alert("Solo administradores pueden registrar préstamos directamente aquí."); return; } // Ajustar si usuarios normales pueden prestar
      if (!this.newLoan.userId || !this.newLoan.bookId || !this.newLoan.dueDate) {
        alert('Por favor, complete todos los campos para el préstamo.'); return;
      }
      const selectedUser = this.users.find(u => u.id === this.newLoan.userId);
      if (selectedUser && selectedUser.isBlocked) {
        alert('Este usuario está bloqueado y no puede realizar préstamos.'); return;
      }
      const bookIndex = this.books.findIndex(b => b.id === this.newLoan.bookId);
      if (bookIndex === -1 || this.books[bookIndex].availableCopies <= 0) {
        alert('El libro seleccionado no está disponible o no existe.'); return;
      }
      const loan = { id: this.generateLoanId(), userId: this.newLoan.userId, bookId: this.newLoan.bookId, loanDate: new Date().toISOString().split('T')[0], dueDate: this.newLoan.dueDate, returnDate: null, status: 'activo' };
      this.loans.push(loan);
      this.books[bookIndex].availableCopies--;
      this.saveDataToLocalStorage();
      alert('Préstamo registrado exitosamente!');
      this.newLoan = { userId: '', bookId: '', dueDate: this.getDefaultDueDate() };
    },
    handleReturnBook(loanId) {
      if (!this.isAdminLoggedIn) { alert("Solo administradores pueden gestionar devoluciones directamente aquí."); return; } // Ajustar
      const loanIndex = this.loans.findIndex(l => l.id === loanId);
      if (loanIndex === -1) { alert('Préstamo no encontrado.'); return; }
      this.loans[loanIndex].status = 'devuelto';
      this.loans[loanIndex].returnDate = new Date().toISOString().split('T')[0];
      const bookIndex = this.books.findIndex(b => b.id === this.loans[loanIndex].bookId);
      if (bookIndex !== -1) { this.books[bookIndex].availableCopies++; }
      this.saveDataToLocalStorage();
      alert('Libro devuelto exitosamente!');
    }
  },
  created() { this.loadDataFromLocalStorage(); this.newLoan.dueDate = this.getDefaultDueDate(); }
};
</script>

<style scoped>
.loan-management { margin: 20px; font-family: sans-serif; }
.admin-only-message { margin: 20px 0; padding: 10px; background-color: #fff3cd; border: 1px solid #ffeeba; color: #856404; border-radius: 3px; text-align: center; }
.loan-form-section, .active-loans-section, .returned-loans-section { margin-bottom: 20px; padding: 15px; border: 1px solid #ccc; border-radius: 5px; }
.loan-form-section div { margin-bottom: 10px; }
.loan-form-section label { display: block; margin-bottom: 5px; font-weight: bold; }
.loan-form-section select, .loan-form-section input[type="date"] { width: calc(100% - 22px); padding: 8px; border: 1px solid #ddd; border-radius: 3px; }
.loan-form-section button { padding: 10px 15px; margin-top: 10px; background-color: #007bff; color: white; border: none; border-radius: 3px; cursor: pointer; }
table { width: 100%; border-collapse: collapse; margin-top: 10px; }
th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
th { background-color: #f2f2f2; }
td button { padding: 5px 10px; cursor: pointer; }
hr { margin-top: 30px; margin-bottom: 30px; }
</style>