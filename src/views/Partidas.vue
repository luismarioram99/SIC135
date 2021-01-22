<template>
  <div class="partidas">
    <v-container class="my-10 mx-5">
      <h1 class="text-h2">Partidas diarias</h1>
      <v-divider></v-divider>
      
      <v-data-iterator
        hide-default-footer
        class="my-5"
        :items="partidas"
        :item-per-page="5"
        no-data-text="Aun no hay partidas creadas."
        :search="busqueda"
        :sort-by="ordenar"
        :sort-desc="descendente"
      >
        <template v-slot:header>
          <v-toolbar class="my-5" elevation="0">
            <v-text-field
              v-model="busqueda"
              clearable
              filled
              rounded
              hide-details
              prepend-inner-icon="mdi-magnify"
              label="Buscar"
            ></v-text-field>
            <v-spacer></v-spacer>
            <v-select
              v-model="ordenar"
              filled
              rounded
              hide-details
              :items="sortBy"
              item-text="text"
              item-value="param"
              prepend-inner-icon="mdi-sort"
              label="Ordenar por"
            ></v-select>
            <v-spacer></v-spacer>
            <v-btn-toggle v-model="descendente" mandatory>
              <v-btn large depressed :value="false">
                <v-icon>mdi-sort-ascending</v-icon>
              </v-btn>
              <v-btn large depressed :value="true">
                <v-icon>mdi-sort-descending</v-icon>
              </v-btn>
            </v-btn-toggle>
            <v-btn
              large
              depressed
              color="green"              
              dark
              class="mx-3"
            >
              <v-icon>mdi-plus</v-icon>
              Crear
            </v-btn>
          </v-toolbar>
        </template>
        <template v-slot:default="{items}">
          <v-slide-group
            multiple
            show-arrows
          >
          <v-slide-item
            v-for="item in items"
            :key="item.id"
          >
            <v-btn
              class="mx-2"
              depressed
              rounded
            >
              {{item.dia}}/{{item.mes}}/{{item.año}}</v-btn>
          </v-slide-item>
          </v-slide-group>
        </template>
      </v-data-iterator>
      
    </v-container>
  </div>
</template>

<script>
import { db } from "../firebase.js";
export default {
  name: "Partidas",
  data() {
    return {
      partidas: [],
      busqueda: "",
      ordenar: "",
      descendente: false,
    };
  },
  firestore: {
    partidas: db.collection("partidas"),
  },
};
</script>

<style scoped>
</style>