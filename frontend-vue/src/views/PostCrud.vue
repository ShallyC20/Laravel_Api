<template>
  <div class="container">
    <h2>Gestión de Usuarios</h2>

    <!-- FORMULARIO -->
    <form @submit.prevent="submitForm">
      <input v-model="form.name" type="text" placeholder="Nombre" required />
      <input v-model="form.email" type="email" placeholder="Correo" required />
      <input v-model="form.password" type="password" placeholder="Contraseña" :required="!form.id"
        @input="validarPassword" />
      <!-- Mensaje de error si la contraseña es muy corta -->
      <p v-if="passwordError" style="color: red; font-size: 0.9em;">
        {{ passwordError }}
      </p>
      <button class="botonRegistrar" type="submit">{{ form.id ? 'Actualizar' : 'Registrar' }}</button>
      <button v-if="form.id" type="button" @click="cancelarEdicion">Cancelar edición</button>
    </form>

    <!-- TABLA DE USUARIOS -->
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre</th>
          <th>Correo</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user.id">
          <td>{{ user.id }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>
            <button @click="editarUsuario(user)">Editar</button>
            <button @click="borrarUsuario(user.id)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios'; // Importamos Axios para hacer peticiones HTTP

export default {
  // data() define las variables reactivas del componente
  data() {
    return {
      users: [], // Aquí se almacenará la lista de usuarios que vienen del backend
      form: {    // Este objeto se enlaza al formulario (v-model)
        id: null,       // Si hay un id, estamos editando; si es null, estamos creando
        name: '',       // Nombre del usuario (input)
        email: '',      // Correo del usuario (input)
        password: ''    // Contraseña (input)
      },
      passwordError: '' // Aquí se guarda el mensaje de error
    };
  },

  // mounted() se ejecuta automáticamente cuando el componente se monta en la vista
  mounted() {
    this.obtenerUsuarios(); // Llama a la función que obtiene la lista de usuarios
  },

  // Aquí definimos todos los métodos que el componente puede ejecutar
  methods: {

    // fetchUsers obtiene los usuarios desde la API (GET)
    async obtenerUsuarios() {
      try {
        const res = await axios.get('http://localhost:8000/api/users');// Llamada HTTP a la API
        this.users = res.data;// Guarda la respuesta en la variable "users"
      } catch (error) {
        console.error('Error al obtener usuarios:', error);
      }
    },

    validarPassword() {
      // Solo validar si estamos creando (no editando)
      if (!this.form.id && this.form.password.length > 0 && this.form.password.length < 6) {
        this.passwordError = 'Contraseña demasiado corta (min: 6 dígitos)';
      } else {
        this.passwordError = '';
      }
    },

    // submitForm maneja tanto creación como edición de usuarios
    async submitForm() {
      // Validar contraseña solo al crear
      if (!this.form.id && this.form.password.length < 6) {
        this.passwordError = 'Contraseña demasiado corta (min: 6 dígitos)';
        return;
      }

      if (this.form.id) {
        // Si hay ID en el formulario, se trata de una edición (PUT)
        await axios.put(`http://localhost:8000/api/users/${this.form.id}`, {
          name: this.form.name,
          email: this.form.email,
          // Si password está vacío, no se envía (para no sobreescribir con vacío)
          password: this.form.password || undefined
        });
      } else {
        // Si no hay ID, se crea un nuevo usuario (POST)
        await axios.post('http://localhost:8000/api/users', this.form);
      }

      // Después de guardar (crear o actualizar):
      this.limpiarForm();     // Limpiamos el formulario
      this.obtenerUsuarios();    // Actualizamos la lista de usuarios
    },

    // editUser llena el formulario con los datos del usuario que se quiere editar
    editarUsuario(user) {
      // Copiamos los datos del usuario al formulario, pero limpiamos la contraseña
      this.form = { ...user, password: '' };
      this.passwordError = '';
    },

    // deleteUser elimina un usuario después de confirmar
    async borrarUsuario(id) {
      if (confirm('¿Estás seguro de eliminar este usuario?')) {
        await axios.delete(`http://localhost:8000/api/users/${id}`); // Llamada DELETE a la API
        this.obtenerUsuarios(); // Actualiza la lista después de eliminar
      }
    },

    // cancelEdicion permite cancelar la edición y volver al modo "crear"
    cancelarEdicion() {
      this.limpiarForm(); // Limpia el formulario
    },

    // limpiarForm limpia todos los campos del formulario y reinicia el modo "crear"
    limpiarForm() {
      this.form = { id: null, name: '', email: '', password: '' };
    }
  }
};
</script>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f0f2f5;
}

.container {
  max-width: 1000px;
  margin: 40px auto;
  background-color: #fc4444b9;
  border-radius: 12px;
  padding: 40px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

/* FORMULARIO */
form {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 30px;
}

input {
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 1rem;
  transition: border-color 0.3s;
}

input:focus {
  outline: none;
  border-color: #dc3545;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s;
}

.botonRegistrar {
  background-color: #dc3545;
  color: #fff;
}

.botonRegistrar:hover {
  background-color: #c82333;
}

button[type="button"] {
  background-color: #6c757d;
  color: #fff;
}

button[type="button"]:hover {
  background-color: #5a6268;
}

/* MENSAJE DE ERROR */
p {
  margin: 0;
}

p[style*="color: red"] {
  font-size: 0.9em;
  font-weight: bold;
}

/* TABLA DE USUARIOS */
table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
}

th,
td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #ffffff7e;
  color: #333;
}

tr:hover {
  background-color: #d50c0c;
}

td button {
  margin-right: 8px;
  padding: 6px 12px;
  font-size: 0.85rem;
}

td button:first-child {
  background-color: #007bff;
  color: white;
}

td button:first-child:hover {
  background-color: #0056b3;
}

td button:last-child {
  background-color: #dc3545;
  color: white;
}

td button:last-child:hover {
  background-color: #c82333;
}

</style>
