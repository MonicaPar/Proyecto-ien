<template>
    <v-app-bar :color="appBarColor">
      <v-app-bar-title>
        <v-btn variant="plain" to="/">
          <template v-slot:prepend>
            <v-img :width="50" aspect-ratio="4/3" cover src="../../assets/images/findTable.png" rounded="circle"></v-img>
          </template>
          SPOT2DINE
        </v-btn>
      </v-app-bar-title>
  
      <v-spacer></v-spacer>
  
      <v-btn class="text-none" stacked to="/">
        <v-icon>mdi-silverware-fork-knife</v-icon> Menú
      </v-btn>
      <v-btn class="text-none" stacked to="/promos"> 
        <v-icon>mdi-tag-multiple</v-icon> Promos
      </v-btn>
      <v-btn class="text-none" stacked to="/reservation"> 
        <v-icon>mdi-calendar-month</v-icon> Reserva ya!
      </v-btn>
      <v-btn class="text-none" stacked to="/detallesReserva" v-if="!isAuthenticated_admin">
        <v-icon> mdi-eye </v-icon> Ver Reservas
      </v-btn>
      <v-btn class="text-none" stacked to="/maintenances" v-if="isAuthenticated_admin">
        <v-icon>mdi-tools</v-icon> Mantenimientos
      </v-btn>
  
  
      <v-spacer></v-spacer>
  
      <div class="text-center" v-if="isAuthenticated">
        <v-btn size="small" variant="flat" color="light-blue-lighten-3" append-icon="mdi-logout" class="me-2"
          @click="logout">
          Cerrar Sesión
        </v-btn>
      </div>
      <div class="text-center" v-else>
        <v-dialog max-width="500">
          <template v-slot:activator="{ props: activatorProps }">
            <v-btn size="small" variant="flat" color="light-blue-lighten-3" append-icon="mdi-login"
              v-bind="activatorProps" class="me-2">
              Inicia Sesión
            </v-btn>
          </template>
          <template v-slot:default="{ isActive }">
            <v-card class="mx-auto" min-width="500">
              <v-card-title :style="'background-color:' + appBarColor" class="mb-2 text-center text-none">
                Inicia Sesión
              </v-card-title>
              <v-list class="pa-3">
                <v-form v-model="form" @submit.prevent="signIn">
                  <v-text-field class="mb-2" v-model="email" :rules="emailRules" label="Correo Electrónico o Usuario"
                    variant="outlined" required clearable></v-text-field>
                  <v-text-field class="mb-2" v-model="password" :readonly="loading" :rules="passRules"
                    :type="showPassword ? 'text' : 'password'" :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                    @click:append="showPassword = !showPassword" label="Contraseña" variant="outlined" clearable
                    required></v-text-field>
                  <v-btn :loading="loading" color="green-lighten-1" size="large" type="submit" variant="elevated" block>
                    Iniciar Sesión
                  </v-btn>
                  <hr class="my-2" />
                  <p class="my-2 text-center" style="color:#424EFC;">
                    ¿No tienes una cuenta?
                  </p>
                  <v-btn size="large" variant="elevated" color="light-blue-lighten-3" block to="/crearCuenta"
                    @click="isActive.value = false">
                    Crear Cuenta
                  </v-btn>
                </v-form>
              </v-list>
            </v-card>
          </template>
        </v-dialog>
      </div>
    </v-app-bar>
  </template>
  
  <script>
  export default {
    data() {
      return {
        email: '',
        password: null,
        loading: false,
        emailRules: [
          value => (value ? true : 'El correo electrónico o usuario es requerido.'),
        ],
        passRules: [
          value => (value ? true : 'La contraseña es requerida.'),
        ],
        showPassword: false,
        form: null,
        appBarColor: '#ffeb3b', // Color inicial de la barra de navegación
      };
    },
    mounted() {
      this.fetchConfiguracion();
      // Cargar el color guardado en localStorage si existe
      
    },
    methods: {
      async fetchConfiguracion() {
        try {
          const response = await fetch('http://127.0.0.1:5000/getConfiguracion');
          if (response.ok) {
            const data = await response.json();
            localStorage.setItem('appBarColor', data.color);
            const savedColor = data.color;
            if (savedColor) {
              this.appBarColor = savedColor;
            }
          }
        } catch (error) {
          console.log(error);
        }
      },
      logout() {
        localStorage.removeItem('token');
        localStorage.removeItem('tipoUser');
        localStorage.removeItem('userId');
        location.reload();
      },
      async signIn() {
              try {
                  this.loading = true;
                  const response = await fetch('http://127.0.0.1:5000/login', {
                      method: 'POST',
                      headers: { 'Content-Type': 'application/json' },
                      body: JSON.stringify({
                          username: this.email,
                          password: this.password,
                      }),
                  });
                  const data = await response.json();
                  if (response.ok && data.token) {
                      localStorage.setItem('tipoUser', data.data.Tipo_User);
                      localStorage.setItem('token', data.token);
                      localStorage.setItem('userId', data.data.Id_User);
                      location.reload();
                  } else {
                      alert(data.message || 'Login failed');
                  }
              } catch (error) {
                  console.error('Error during login:', error);
                  alert('An error occurred during login.');
              } finally {
                  this.loading = false;
              }
          },
    },
    computed: {
      isAuthenticated() {
        return !!localStorage.getItem('token');
      },
  
      isAuthenticated_admin() {
        return localStorage.getItem('tipoUser') == 0;
      }
    },
  };
  </script>