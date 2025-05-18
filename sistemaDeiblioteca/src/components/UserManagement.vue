<template>
  <div class="user-management">
    <h2>Gestión de Usuarios</h2>
    <div class="user-form" v-if="isAdminLoggedIn">
      <h3>{{ editingUser ? 'Editar Usuario' : 'Añadir Nuevo Usuario' }}</h3>
      <form @submit.prevent="handleSubmitUser">
        <div>
          <label for="name">Nombre:</label>
          <input type="text" id="name" v-model="currentUser.name" required />
        </div>
        <div>
          <label for="email">Email:</label>
          <input type="email" id="email" v-model="currentUser.email" required />
        </div>
        <div>
          <label for="type">Tipo:</label>
          <select id="type" v-model="currentUser.type" required>
            <option value="estudiante">Estudiante</option>
            <option value="docente">Docente</option>
            <option value="administrador">Administrador</option>
          </select>
        </div>
        <div v-if="editingUser">
            <label for="user-blocked" style="display: inline-block; margin-right: 5px;">Bloqueado:</label>
            <input type="checkbox" id="user-blocked" v-model="currentUser.isBlocked" style="width: auto; vertical-align: middle;" />
        </div>
        <button type="submit">{{ editingUser ? 'Actualizar' : 'Guardar' }}</button>
        <button type="button" @click="cancelEdit" v-if="editingUser">Cancelar</button>
      </form>
    </div>
    <div v-else-if="!isAdminLoggedIn && users.length > 0" class="admin-only-message">
      <p><em>La adición y edición de usuarios está restringida a administradores.</em></p>
    </div>
    <hr />
    <div class="user-table card shadow-sm p-3 mb-4 bg-light border-0"> <h3>Lista de Usuarios</h3>
      <h3>Lista de Usuarios</h3>
       <table class="table table-striped table-hover table-bordered table-sm">
        <thead class="table-dark">
          <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Email</th>
            <th>Tipo</th>
            <th>Estado</th>
            <th v-if="isAdminLoggedIn">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="users.length === 0">
            <td :colspan="isAdminLoggedIn ? 6 : 5" style="text-align: center;">No hay usuarios registrados.</td>
          </tr>
          <tr v-for="user in users" :key="user.id" :class="{ 'blocked-user': user.isBlocked }">
            <td>{{ user.id }}</td>
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
            <td>{{ user.type }}</td>
            <td>{{ user.isBlocked ? 'Bloqueado' : 'Activo' }}</td>
            <td v-if="isAdminLoggedIn">
              <button class="btn btn-warning btn-sm me-1" @click="editUser(user)">Editar</button>
              <button class="btn btn-danger btn-sm me-1" @click="deleteUser(user.id)">Eliminar</button>
              <button class="btn btn-info btn-sm" @click="toggleBlockUser(user.id)">
                {{ user.isBlocked ? 'Desbloquear' : 'Bloquear' }}
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserManagement',
  props: {
    isAdminLoggedIn: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      users: [],
      currentUser: {
        id: null, name: '', email: '', type: 'estudiante', isBlocked: false
      },
      editingUser: null,
      nextUserId: 1
    };
  },
  methods: {
    initializeUser(user) {
      return { ...user, isBlocked: user.isBlocked || false };
    },
    generateId() {
      return 'u' + this.nextUserId++;
    },
    handleSubmitUser() {
      if (!this.isAdminLoggedIn) { alert("Acción no permitida."); return; }
      if (!this.currentUser.name || !this.currentUser.email || !this.currentUser.type) {
        alert('Por favor, completa todos los campos requeridos (Nombre, Email, Tipo).');
        return;
      }
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(this.currentUser.email)) {
        alert('Por favor, ingresa un formato de email válido.');
        return;
      }

      if (this.editingUser) {
        const index = this.users.findIndex(u => u.id === this.editingUser.id);
        if (index !== -1) {
          this.users.splice(index, 1, this.initializeUser({ ...this.currentUser, id: this.editingUser.id }));
        }
      } else {
        const newUser = this.initializeUser({ ...this.currentUser, id: this.generateId() });
        this.users.push(newUser);
      }
      this.resetForm();
    },
    editUser(user) {
      if (!this.isAdminLoggedIn) return;
      this.editingUser = this.initializeUser({ ...user });
      this.currentUser = this.initializeUser({ ...user });
    },
    deleteUser(userId) {
      if (!this.isAdminLoggedIn) return;
      if (confirm('¿Estás seguro de que quieres eliminar este usuario?')) {
        this.users = this.users.filter(user => user.id !== userId);
        if (this.editingUser && this.editingUser.id === userId) {
          this.resetForm();
        }
      }
    },
    resetForm() {
      this.currentUser = { id: null, name: '', email: '', type: 'estudiante', isBlocked: false };
      this.editingUser = null;
    },
    cancelEdit() { this.resetForm(); },
    toggleBlockUser(userId) {
      if (!this.isAdminLoggedIn) { alert("Acción no permitida."); return; }
      const userIndex = this.users.findIndex(user => user.id === userId);
      if (userIndex !== -1) {
        this.users[userIndex].isBlocked = !this.users[userIndex].isBlocked;
      }
    }
  },
  mounted() {
    const storedUsers = localStorage.getItem('biblioteca_users');
    if (storedUsers) {
      const parsedUsers = JSON.parse(storedUsers);
      this.users = parsedUsers.map(user => this.initializeUser(user));
      if (this.users.length > 0) {
        const maxIdNum = this.users.reduce((max, user) => {
            const idNum = parseInt(user.id.substring(1));
            return !isNaN(idNum) && idNum > max ? idNum : max;
        }, 0);
        this.nextUserId = maxIdNum + 1;
      } else { this.nextUserId = 1; }
    } else {
      // Datos iniciales si localStorage está vacío
      this.users = [
        this.initializeUser({ id: this.generateId(), name: 'Admin Ejemplo', email: 'admin@example.com', type: 'administrador' }),
        this.initializeUser({ id: this.generateId(), name: 'Estudiante Ejemplo', email: 'estudiante@example.com', type: 'estudiante', isBlocked: true })
      ];
    }
  },
  watch: {
    users: {
      handler(newUsers) {
        localStorage.setItem('biblioteca_users', JSON.stringify(newUsers));
      },
      deep: true
    }
  }
};
</script>

<style scoped>

</style>