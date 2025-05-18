<template>
  <div class="loan-management">
    <h2>Gestión de Préstamos</h2>

    <div class="loan-form-section" v-if="isAdminLoggedIn"> 
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
    <div class="active-reservations-section">
  <h3>Reservas Activas</h3>
  <table v-if="activeReservations.length > 0">
    <thead>
      <tr>
        <th>Libro</th>
        <th>Usuario</th>
        <th>Fecha Reserva</th>
        <th v-if="isAdminLoggedIn">Acciones</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="reservation in activeReservations" :key="reservation.id">
        <td>{{ reservation.bookTitle || getBookTitle(reservation.bookId) }}</td>
        <td>{{ reservation.userName || getUserName(reservation.userId) }}</td>
        <td>{{ formatDate(reservation.reservationDate) }}</td>
        <td v-if="isAdminLoggedIn">
        
          <button @click="handleCancelReservation(reservation.id)">Cancelar Reserva</button>
        </td>
      </tr>
    </tbody>
  </table>
  <p v-else>No hay reservas activas.</p>
</div>
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
  props: {
    isAdminLoggedIn: {
      type: Boolean,
      default: false
    }
    // currentUserId: { type: String, default: null } // Ejemplo si necesitaras el ID del usuario normal
  },
  data() {
    return {
      loans: [],
      users: [],
      books: [],
      newLoan: { 
        userId: '', 
        bookId: '', 
        dueDate: this.getDefaultDueDate() 
      },
      nextLoanIdCounter: 1, 
      reservations: [],
      // nextReservationIdCounter: 1, // No es necesario si generamos IDs de reserva con Date.now()
      localStorageKeys: {
        users: 'biblioteca_users',
        books: 'biblioteca_books_vue_project',
        loans: 'biblioteca_loans_vue_project',
        reservations: 'biblioteca_reservations_vue_project'
      }
    };
  },
  computed: {
    availableUsersForLoan() {
      return this.users.filter(user => !user.isBlocked);
    },
    activeReservations() {
      return this.reservations.filter(res => res.status === 'activa');
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
    activeLoansForCurrentUser() { // Ejemplo, necesitaría currentUserId
      return []; 
    },
    returnedLoansForCurrentUser() { // Ejemplo, necesitaría currentUserId
      return [];
    },
    loansToDisplay() {
      if (this.isAdminLoggedIn) {
        return { active: this.allActiveLoans, returned: this.allReturnedLoans };
      }
      return { active: this.activeLoansForCurrentUser, returned: this.returnedLoansForCurrentUser };
    }
  },
  methods: { // INICIO DEL BLOQUE DE MÉTODOS
    getDefaultDueDate() {
      const today = new Date();
      today.setDate(today.getDate() + 7);
      return today.toISOString().split('T')[0];
    },

    formatDate(dateString) {
      if (!dateString) return 'N/A';
      const date = new Date(dateString);
      return date.toLocaleDateString('es-CO', { year: 'numeric', month: 'long', day: 'numeric' });
    },

    loadDataFromLocalStorage() {
      console.log('LoanManagement: Cargando datos desde localStorage...');
      // Cargar Usuarios
      const storedUsers = localStorage.getItem(this.localStorageKeys.users);
      if (storedUsers) {
        try {
          this.users = JSON.parse(storedUsers).map(u => ({ ...u, isBlocked: u.isBlocked || false }));
        } catch (e) { console.error('Error parseando usuarios:', e); this.users = []; }
      } else { this.users = []; }

      // Cargar Libros
      const storedBooks = localStorage.getItem(this.localStorageKeys.books);
      if (storedBooks) {
        try { this.books = JSON.parse(storedBooks); }
        catch (e) { console.error('Error parseando libros:', e); this.books = []; }
      } else { this.books = []; }
      
      // Cargar Préstamos
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

      // Cargar Reservas
      const storedReservations = localStorage.getItem(this.localStorageKeys.reservations);
      if (storedReservations) {
        try { this.reservations = JSON.parse(storedReservations); }
        catch (e) { console.error('Error parseando reservaciones:', e); this.reservations = []; }
      } else { this.reservations = []; }
      console.log('LoanManagement: Datos cargados.');
    },

    saveDataToLocalStorage() {
      localStorage.setItem(this.localStorageKeys.users, JSON.stringify(this.users));
      localStorage.setItem(this.localStorageKeys.books, JSON.stringify(this.books));
      localStorage.setItem(this.localStorageKeys.loans, JSON.stringify(this.loans));
      localStorage.setItem(this.localStorageKeys.reservations, JSON.stringify(this.reservations));
      console.log('LoanManagement: Datos guardados en localStorage.');
    },

    generateLoanId() {
      return 'ln' + this.nextLoanIdCounter++;
    },

    getBookTitle(bookId) {
      const book = this.books.find(b => b.id === bookId);
      return book ? book.title : 'Libro Desconocido';
    },

    getUserName(userId) {
      const user = this.users.find(u => u.id === userId);
      return user ? user.name : 'Usuario Desconocido';
    },

    handleRegisterLoan() {
      if (!this.isAdminLoggedIn) { alert("Solo administradores pueden registrar préstamos."); return; }
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
      const loan = { 
        id: this.generateLoanId(), 
        userId: this.newLoan.userId, 
        bookId: this.newLoan.bookId, 
        loanDate: new Date().toISOString().split('T')[0], 
        dueDate: this.newLoan.dueDate, 
        returnDate: null, 
        status: 'activo' 
      };
      this.loans.push(loan);
      this.books[bookIndex].availableCopies--;
      this.saveDataToLocalStorage();
      alert('Préstamo registrado exitosamente!');
      this.newLoan = { userId: '', bookId: '', dueDate: this.getDefaultDueDate() };
    },

    handleReturnBook(loanId) {
      if (!this.isAdminLoggedIn) { alert("Solo administradores pueden gestionar devoluciones."); return; }
      const loanIndex = this.loans.findIndex(l => l.id === loanId);
      if (loanIndex === -1) { alert('Préstamo no encontrado.'); return; }
      
      this.loans[loanIndex].status = 'devuelto';
      this.loans[loanIndex].returnDate = new Date().toISOString().split('T')[0];
      
      const bookIndex = this.books.findIndex(b => b.id === this.loans[loanIndex].bookId);
      if (bookIndex !== -1) {
        this.books[bookIndex].availableCopies++;
      }
      this.saveDataToLocalStorage();
      alert('Libro devuelto exitosamente!');
    },

    handleCancelReservation(reservationId) {
      if (!this.isAdminLoggedIn) {
        alert("Solo administradores pueden cancelar reservas."); return;
      }
      if (confirm("¿Está seguro de que desea cancelar esta reserva?")) {
        this.reservations = this.reservations.filter(res => res.id !== reservationId);
        this.saveDataToLocalStorage();
        alert("Reserva cancelada.");
      }
    }
  }, // FIN DEL BLOQUE DE MÉTODOS
  created() { 
    this.loadDataFromLocalStorage(); 
    this.newLoan.dueDate = this.getDefaultDueDate(); 
  }
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