<template>
  <v-app>
        <v-navigation-drawer app permanent>
          <v-list>
            <v-list-item>
              <router-link to="/">
                <v-img src="./assets/contableX-logo.png"></v-img>
              </router-link>
            </v-list-item>

            <v-list-item>
              <v-list-item-content>
                <v-list-item-title class="title"> ContableX </v-list-item-title>
                <v-list-item-subtitle>Tu sistema contable</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>

          <v-divider></v-divider>

          <v-list nav dense>
            <v-list-item link to="/cuentas">
              <v-list-item-icon>
                <v-icon>mdi-clipboard-list</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Catálogo de cuentas</v-list-item-title>
            </v-list-item>
            <v-list-item link to="/partidas">
              <v-list-item-icon>
                <v-icon>mdi-file-document-edit</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Partidas diarias</v-list-item-title>
            </v-list-item>
           
            <v-list-item link to="/empleados">
              <v-list-item-icon>
                <v-icon>mdi-account-group</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Planilla de empleados</v-list-item-title>
            </v-list-item>
            <v-list-item link>
              <v-list-item-icon>
                <v-icon>mdi-scale-balance</v-icon>
              </v-list-item-icon>
              <v-list-item-title>Balance de comprobación</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-navigation-drawer>

        <v-main>
          <router-view></router-view>
        </v-main>
  </v-app>
</template>

<script>

export default {
  name: "App",

  components: {
  },

  data: () => ({
    
  }),
};
</script>
