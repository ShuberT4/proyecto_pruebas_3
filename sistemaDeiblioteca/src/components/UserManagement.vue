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

    <div class="user-table">
      <h3>Lista de Usuarios</h3>
      <table border="1" style="width: 100%; border-collapse: collapse;">
        <thead>
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
              <button @click="editUser(user)">Editar</button>
              <button @click="deleteUser(user.id)">Eliminar</button>
              <button @click="toggleBlockUser(user.id)">
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
.user-management { margin: 20px; font-family: sans-serif; }
.admin-only-message { margin: 20px 0; padding: 10px; background-color: #fff3cd; border: 1px solid #ffeeba; color: #856404; border-radius: 3px; text-align: center; }
.blocked-user td { color: #999; text-decoration: line-through; }
.user-form { margin-bottom: 20px; padding: 15px; border: 1px solid #ccc; border-radius: 5px; }
.user-form div { margin-bottom: 10px; }
.user-form label { display: block; margin-bottom: 5px; font-weight: bold; }
.user-form input[type="text"],
.user-form input[type="email"],
.user-form select { width: calc(100% - 22px); padding: 8px; border: 1px solid #ddd; border-radius: 3px; }
.user-form input[type="checkbox"] { width: auto; margin-left: 5px; vertical-align: middle; }
.user-form button { padding: 10px 15px; margin-right: 10px; background-color: #007bff; color: white; border: none; border-radius: 3px; cursor: pointer; }
.user-form button[type="button"] { background-color: #6c757d; }
.user-table table { margin-top: 10px; width:100%; border-collapse:collapse; }
.user-table th, .user-table td { padding: 8px; text-align: left; border: 1px solid #ddd; }
.user-table th { background-color: #f2f2f2; }
.user-table button { margin: 0 2px; padding: 5px 8px; font-size:0.9em; }
hr { margin-top: 30px; margin-bottom: 30px; }
</style>