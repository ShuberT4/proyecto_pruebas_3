<template>
  <template>
  <div class="catalog-management"> {/* INICIO DEL DIV PRINCIPAL */}
    <h2>Catálogo de Recursos</h2>

    {/* MODAL DE RESERVA - AHORA CON ESTRUCTURA CORRECTA */}
    <div v-if="showReserveModal" class="modal-overlay">
      <div class="modal-content"> {/* Contenedor interno del modal */}
        <h3>Reservar Libro: {{ bookToReserve?.title }}</h3>
        <p v-if="bookToReserve">Este libro ({{ bookToReserve.title }}) no tiene copias disponibles actualmente.</p>
        <div>
          <label for="reserve-user">Reservar para el usuario:</label>
          <select id="reserve-user" v-model="selectedUserIdForReservation" required>
            <option disabled value="">Seleccione un usuario</option>
            <option v-for="userInModal in users" :key="userInModal.id" :value="userInModal.id">
              {{ userInModal.name }} ({{ userInModal.email }})
            </option>
          </select>
        </div>
        <div class="modal-actions">
          <button @click="confirmReservation">Confirmar Reserva</button>
          <button @click="closeReserveModal">Cancelar</button>
        </div>
      </div>
    </div>
    {/* FIN DEL MODAL DE RESERVA */}

    {/* FORMULARIO DE LIBROS (CRUD) */}
    <div class="book-form-section" v-if="isAdminLoggedIn">
      <h3>{{ editingBook ? 'Editar Libro' : 'Añadir Nuevo Libro' }}</h3>
      <form @submit.prevent="handleSubmitBook">
        <div> <label for="book-title">Título:</label> <input type="text" id="book-title" v-model="currentBook.title" required /> </div>
        <div> <label for="book-author">Autor:</label> <input type="text" id="book-author" v-model="currentBook.author" required /> </div>
        <div> <label for="book-isbn">ISBN:</label> <input type="text" id="book-isbn" v-model="currentBook.isbn" required /> </div>
        <div> <label for="book-genre">Género:</label> <input type="text" id="book-genre" v-model="currentBook.genre" /> </div>
        <div> <label for="book-description">Descripción:</label> <textarea id="book-description" v-model="currentBook.description"></textarea> </div>
        <div> <label for="book-coverImage">URL de Portada:</label> <input type="url" id="book-coverImage" v-model="currentBook.coverImage" /> </div>
        <div> <label for="book-totalCopies">Copias Totales:</label> <input type="number" id="book-totalCopies" v-model.number="currentBook.totalCopies" min="0" required /> </div>
        <button type="submit">{{ editingBook ? 'Actualizar Libro' : 'Guardar Libro' }}</button>
        <button type="button" @click="resetBookForm" v-if="editingBook">Cancelar Edición</button>
      </form>
    </div>
    <div v-else class="admin-only-message">
        <p><em>La adición y edición de libros está restringida a administradores.</em></p>
    </div>

    <hr />

    {/* FILTROS */}
    <div class="filters">
      <input type="text" v-model="searchTerm" placeholder="Buscar por título, autor, ISBN..." />
      <select v-model="selectedGenre">
        <option value="">Todos los Géneros</option>
        <option v-for="genreValue in uniqueGenres" :key="genreValue" :value="genreValue">{{ genreValue }}</option>
      </select>
    </div>

    {/* LISTA DE LIBROS */}
    <div class="book-list">
      <div v-if="filteredBooks.length === 0" class="no-books">
        No se encontraron libros con los criterios seleccionados.
      </div>
      <div v-for="book in filteredBooks" :key="book.id" class="book-item">
        <img :src="book.coverImage || defaultCoverImage" :alt="book.title" class="book-cover-placeholder" />
        <h3>{{ book.title }}</h3>
        <p><strong>Autor:</strong> {{ book.author }}</p>
        <p><strong>ISBN:</strong> {{ book.isbn }}</p>
        <p><strong>Género:</strong> {{ book.genre }}</p>
        <p><strong>Disponibles:</strong> {{ book.availableCopies }} / {{ book.totalCopies }}</p>
        <p class="book-description-small">{{ book.description }}</p>
        <div class="book-actions">
          <button v-if="isAdminLoggedIn" @click="prepareEditBook(book)">Editar</button>
          <button v-if="isAdminLoggedIn" @click="removeBook(book.id)">Eliminar</button>
          {/* Botón de Reserva */}
          <button
            v-if="isAdminLoggedIn && book.availableCopies === 0"
            @click="openReserveModal(book)"
            class="reserve-button"
          >
            Reservar
          </button>
        </div>
      </div>
    </div>

  </div> {/* CIERRE DEL DIV PRINCIPAL 'catalog-management' */}
</template>
</template>

<script>
export default {
  name: 'CatalogView',
  props: {
    isAdminLoggedIn: {
      type: Boolean,
      default: false
    }
  },
  data() {
  return {
    books: [],
    currentBook: { id: null, title: '', author: '', isbn: '', genre: '', description: '', coverImage: '', totalCopies: 0, availableCopies: 0 },
    editingBook: null,
    nextBookIdCounter: 1,
    searchTerm: '',
    selectedGenre: '',
    defaultCoverImage: 'https://placehold.co/150x220/cccccc/969696?text=Libro', // Coma añadida

    // Para Reservas
    users: [], // Corregido de user[] a users: [], y con coma
    showReserveModal: false, // Corregido el typo (era showReseveModal) y con coma
    bookToReserve: null,     // Coma añadida
    selectedUserIdForReservation: '', // Coma añadida
    
    // Claves de LocalStorage (asegúrate de que estén todas las que necesitas)
    localStorageUserKey: 'biblioteca_users', // Añadida si la usas en loadUsersForReservation
    localStorageReservationsKey: 'biblioteca_reservations_vue_project'
  };
},
  computed: {
    uniqueGenres() {
      const genres = this.books.map(book => book.genre).filter(genre => genre);
      return [...new Set(genres)];
    },
    filteredBooks() {
      let filtered = this.books;
      if (this.searchTerm) {
        const lowerSearchTerm = this.searchTerm.toLowerCase();
        filtered = filtered.filter(book =>
          book.title.toLowerCase().includes(lowerSearchTerm) ||
          book.author.toLowerCase().includes(lowerSearchTerm) ||
          book.isbn.toLowerCase().includes(lowerSearchTerm)
        );
      }
      if (this.selectedGenre) {
        filtered = filtered.filter(book => book.genre === this.selectedGenre);
      }
      return filtered;
    }
  },
  methods: {
    generateBookId() { return 'b' + this.nextBookIdCounter++; },
     loadUsersForReservation() { // Cargar usuarios para el selector
    const storedUsers = localStorage.getItem(this.localStorageUserKey);
    if (storedUsers) {
      this.users = JSON.parse(storedUsers).filter(user => !user.isBlocked); // Opcional: no permitir reservar a bloqueados
    } else {
      this.users = [];
    }
  },
  openReserveModal(book) {
    if (!this.isAdminLoggedIn) return;
    this.loadUsersForReservation(); // Cargar/actualizar lista de usuarios
    this.bookToReserve = book;
    this.selectedUserIdForReservation = ''; // Resetear selección
    this.showReserveModal = true;
  },
   closeReserveModal() {
    this.showReserveModal = false;
    this.bookToReserve = null;
    this.selectedUserIdForReservation = '';
  },
    confirmReservation() {
    if (!this.selectedUserIdForReservation || !this.bookToReserve) {
      alert("Por favor, seleccione un usuario para la reserva.");
      return;
    }

    let reservations = JSON.parse(localStorage.getItem(this.localStorageReservationsKey)) || [];
    
    // Opcional: Verificar si el usuario ya tiene una reserva activa para este libro
    const existingReservation = reservations.find(
      res => res.userId === this.selectedUserIdForReservation && 
             res.bookId === this.bookToReserve.id &&
             res.status === 'activa'
    );
    if (existingReservation) {
      alert("Este usuario ya tiene una reserva activa para este libro.");
      this.closeReserveModal();
      return;
    }

    const newReservation = {
      id: 'r' + Date.now() + Math.random().toString(16).slice(2), // ID único simple
      userId: this.selectedUserIdForReservation,
      bookId: this.bookToReserve.id,
      bookTitle: this.bookToReserve.title, // Guardar título para fácil visualización
      userName: this.users.find(u => u.id === this.selectedUserIdForReservation)?.name || 'Desconocido', // Guardar nombre
      reservationDate: new Date().toISOString().split('T')[0],
      status: 'activa' // 'activa', 'cancelada', 'completada' (cuando se convierte en préstamo)
    };

    reservations.push(newReservation);
    localStorage.setItem(this.localStorageReservationsKey, JSON.stringify(reservations));
    
    alert(`Reserva para "${this.bookToReserve.title}" a nombre del usuario seleccionado registrada.`);
    this.closeReserveModal();
  }
},

mounted() {
  // ... (tu carga de libros existente) ...
  const storedBooks = localStorage.getItem('biblioteca_books_vue_project'); /* ... */
  // Cargar usuarios también al montar si quieres que el selector esté listo siempre,
  // o solo al abrir el modal como en openReserveModal().
  // this.loadUsersForReservation(); 
},

    handleSubmitBook() {
      if (!this.isAdminLoggedIn) { alert("Acción no permitida."); return; }
      if (!this.currentBook.title || !this.currentBook.author || !this.currentBook.isbn) {
        alert('Título, Autor e ISBN son campos requeridos.'); return;
      }
      const isbnToCheck = this.currentBook.isbn.trim();
      if (this.editingBook) {
        if (this.currentBook.isbn !== this.editingBook.isbn) {
          const existingBookWithNewIsbn = this.books.find(book => book.isbn === isbnToCheck && book.id !== this.editingBook.id);
          if (existingBookWithNewIsbn) { alert('Error: Ya existe otro libro con el ISBN ingresado.'); return; }
        }
        const index = this.books.findIndex(b => b.id === this.editingBook.id);
        if (index !== -1) {
          let newAvailableCopies = this.books[index].availableCopies;
          if (this.currentBook.totalCopies !== this.books[index].totalCopies) {
            const diffCopies = this.currentBook.totalCopies - this.books[index].totalCopies;
            newAvailableCopies = Math.max(0, this.books[index].availableCopies + diffCopies);
            newAvailableCopies = Math.min(newAvailableCopies, this.currentBook.totalCopies);
          }
          this.books.splice(index, 1, { ...this.currentBook, isbn: isbnToCheck, id: this.editingBook.id, availableCopies: newAvailableCopies });
        }
      } else {
        const existingBook = this.books.find(book => book.isbn === isbnToCheck);
        if (existingBook) { alert('Error: Ya existe un libro con el ISBN ingresado.'); return; }
        const newBook = { ...this.currentBook, isbn: isbnToCheck, id: this.generateBookId(), availableCopies: this.currentBook.totalCopies };
        this.books.push(newBook);
      }
      this.resetBookForm();
    },
    prepareEditBook(book) {
      if (!this.isAdminLoggedIn) return;
      this.editingBook = { ...book }; this.currentBook = { ...book };
    },
    removeBook(bookId) {
      if (!this.isAdminLoggedIn) return;
      if (confirm('¿Estás seguro de que quieres eliminar este libro?')) {
        this.books = this.books.filter(book => book.id !== bookId);
        if (this.editingBook && this.editingBook.id === bookId) { this.resetBookForm(); }
      }
    },
    resetBookForm() {
      this.currentBook = { id: null, title: '', author: '', isbn: '', genre: '', description: '', coverImage: '', totalCopies: 0, availableCopies: 0 };
      this.editingBook = null;
  },

  mounted() {
    const storedBooks = localStorage.getItem('biblioteca_books_vue_project');
    if (storedBooks) {
      this.books = JSON.parse(storedBooks);
      if (this.books.length > 0) {
        const maxIdNum = this.books.reduce((max, book) => {
            const idNum = parseInt(book.id.substring(1));
            return !isNaN(idNum) && idNum > max ? idNum : max;
        }, 0);
        this.nextBookIdCounter = maxIdNum + 1;
      } else { this.nextBookIdCounter = 1; }
    } else {
      // Datos iniciales si localStorage está vacío
       this.books = [
        { id: this.generateBookId(), title: 'El Gran Gatsby', author: 'F. Scott Fitzgerald', isbn: '978-0743273565', genre: 'Ficción Clásica', description: 'Una historia sobre la decadencia...', coverImage: 'https://placehold.co/150x220/E8D5C4/A27B5C?text=El+Gran+Gatsby', totalCopies: 5, availableCopies: 3 },
        { id: this.generateBookId(), title: '1984', author: 'George Orwell', isbn: '978-0451524935', genre: 'Ciencia Ficción Distópica', description: 'Una visión sombría...', coverImage: 'https://placehold.co/150x220/7D7C7C/F1F1F1?text=1984', totalCopies: 3, availableCopies: 1 }
      ];
    }
  },
  watch: {
    books: {
      handler(newBooks) {
        localStorage.setItem('biblioteca_books_vue_project', JSON.stringify(newBooks));
      },
      deep: true
    }
  }
};
</script>

<style scoped>
.catalog-management { margin: 20px; font-family: sans-serif; }
.admin-only-message { margin: 20px 0; padding: 10px; background-color: #fff3cd; border: 1px solid #ffeeba; color: #856404; border-radius: 3px; text-align: center; }
.book-form-section { margin-bottom: 20px; padding: 15px; border: 1px solid #ccc; border-radius: 5px; }
.book-form-section div { margin-bottom: 10px; }
.book-form-section label { display: block; margin-bottom: 5px; font-weight: bold; }
.book-form-section input[type="text"],
.book-form-section input[type="url"],
.book-form-section input[type="number"],
.book-form-section textarea { width: calc(100% - 22px); padding: 8px; border: 1px solid #ddd; border-radius: 3px; }
.book-form-section textarea { min-height: 60px; }
.book-form-section button { padding: 10px 15px; margin-right: 10px; background-color: #28a745; color: white; border: none; border-radius: 3px; cursor: pointer; }
.book-form-section button[type="button"] { background-color: #6c757d; }

.filters { margin-bottom: 20px; display: flex; gap: 10px; }
.filters input[type="text"] { padding: 8px; border: 1px solid #ddd; border-radius: 3px; flex-grow: 1; }
.filters select { padding: 8px; border: 1px solid #ddd; border-radius: 3px; }

.book-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
.book-item { border: 1px solid #ccc; padding: 15px; border-radius: 5px; text-align: center; display: flex; flex-direction: column; justify-content: space-between; }
.book-cover-placeholder { width: 150px; height: 220px; background-color: #eee; display: block; margin: 0 auto 10px auto; object-fit: cover; }
.book-item h3 { margin-top: 0; font-size: 1.2em; }
.book-item p { font-size: 0.9em; margin: 4px 0; }
.book-description-small { font-size: 0.8em; color: #555; height: 4.8em; overflow: hidden; text-overflow: ellipsis; display: -webkit-box; -webkit-line-clamp: 4; -webkit-box-orient: vertical; }
.book-actions { margin-top: 15px; }
.book-actions button { margin: 0 5px; padding: 6px 10px; font-size: 0.9em; }
.no-books { text-align: center; padding: 20px; width: 100%; grid-column: 1 / -1; }
hr { margin-top: 30px; margin-bottom: 30px; }
/* Añadir a tus estilos existentes en CatalogView.vue */
.reserve-button {
  background-color: #ffc107; /* Amarillo para reserva */
  color: #212529;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
.modal-content {
  background-color: white;
  padding: 20px 30px;
  border-radius: 5px;
  min-width: 300px;
  max-width: 500px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}
.modal-content h3 { margin-top: 0; }
.modal-content label { display: block; margin-bottom: 5px; font-weight: bold; }
.modal-content select { width: 100%; padding: 8px; margin-bottom: 15px; border: 1px solid #ddd; border-radius: 3px; box-sizing: border-box; }
.modal-actions button {
  padding: 10px 15px;
  margin-right: 10px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}
.modal-actions button:first-of-type { background-color: #28a745; color: white; }
.modal-actions button:last-of-type { background-color: #6c757d; color: white; }
</style>