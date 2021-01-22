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
            <v-btn large depressed color="green" dark class="mx-3">
              <v-icon>mdi-plus</v-icon>
              Crear
            </v-btn>
          </v-toolbar>
        </template>
        <template v-slot:default="{ items }">
          <v-slide-group multiple show-arrows>
            <v-slide-item v-for="item in items" :key="item.id">
              <v-btn class="mx-2" depressed rounded @click="selected = item">
                {{ item.dia }}/{{ item.mes }}/{{ item.año }}</v-btn
              >
            </v-slide-item>
          </v-slide-group>
        </template>
      </v-data-iterator>

      <v-card v-if="selected" class="my-5 mx-5" elevation="15">
        <v-form ref="form">
          <v-card-title class="w-100">
            <div class="w-100-2">
              <h3 class="text-h4">
                {{ selected.dia }}/{{ selected.mes }}/{{ selected.año }}
              </h3>
              <v-btn color="deep-purple" icon>
                <v-icon> mdi-calendar </v-icon>
              </v-btn>
            </div>

            <v-btn class="to-right" color="red" icon>
              <v-icon> mdi-delete </v-icon>
            </v-btn>
          </v-card-title>
          <v-divider></v-divider>
          <v-card-text>
            <h4 class="text-h6">Movimientos</h4>
            <v-data-iterator
              class="my-3"
              :items="selected.movimientos"
              hide-default-footer
              hide-default-header
              no-data-text="Agregue movimientos."
            >
              <template v-slot:default="{ items }">
                <v-row
                  class="justify-space-between"
                  v-for="(movimiento, index) in items"
                  :key="index"
                >
                  <v-col cols="6">
                    <v-select
                      filled
                      label="Cuenta"
                      :rules="rules"
                      :items="cuentas"
                      v-model="movimiento.categoria"
                    >
                      <template v-slot:selection="{ item }">
                        {{ item.numero }} -
                        {{
                          item.nombre.length > 30
                            ? item.nombre.substr(0, 30) + "..."
                            : item.nombre
                        }}
                      </template>
                      <template v-slot:item="{ item }">
                        {{ item.numero }} - {{ item.nombre }}
                      </template>
                    </v-select>
                  </v-col>
                  <v-col cols="3">
                    <v-text-field
                      filled
                      type="number"
                      v-model="movimiento.debe"
                      min="0"
                      label="Debe"
                      prefix="$"
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="3">
                    <v-text-field
                      filled
                      type="number"
                      v-model="movimiento.haber"
                      min="0"
                      label="Haber"
                      prefix="$"
                    >
                    </v-text-field>
                  </v-col>
                </v-row>
              </template>
            </v-data-iterator>
            <center>
              <v-btn icon color="green" @click="crearMovimiento()">
                <v-icon>mdi-plus-circle</v-icon>
              </v-btn>
            </center>
          </v-card-text>
        </v-form>
      </v-card>
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
      cuentas: [],
      busqueda: "",
      ordenar: "",
      sortBy: [],
      descendente: false,
      selected: null,
      rules: [(v) => !!v || "Debe llenar este campo."],
    };
  },
  methods: {
    crearMovimiento() {
      console.log("crear movimiento");
      if (!this.selected.movimientos)
        this.selected = { ...this.selected, movimientos: [] };

      this.selected.movimientos.push({
        categoria: null,
        debe: 0,
        haber: 0,
      });

      console.log(this.selected);
    },
  },
  firestore: {
    cuentas: db.collection("cuentas"),
    partidas: db.collection("partidas"),
  },
};
</script>

<style scoped>
.w-100 {
  width: 100% !important;
  display: flex;
  flex-direction: row;
  align-content: center;
  justify-content: space-between;
}
.w-100-2 {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}
.to-right {
  justify-self: right;
}
.bg-red {
  background: red;
}
</style>