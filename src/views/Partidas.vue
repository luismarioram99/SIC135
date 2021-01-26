<template>
  <div class="partidas">
    <v-container class="my-10 mx-5">
      <h1 class="text-h2">Partidas diarias</h1>
      <v-divider></v-divider>

      <v-data-iterator
        
        class="my-5"
        :items="partidas"
        :items-per-page="-1"
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
          
            <v-spacer></v-spacer>
            <v-btn-toggle v-model="descendente" mandatory>
              <v-btn large depressed :value="false">
                <v-icon>mdi-sort-ascending</v-icon>
              </v-btn>
              <v-btn large depressed :value="true">
                <v-icon>mdi-sort-descending</v-icon>
              </v-btn>
            </v-btn-toggle>
            <v-btn large depressed color="green" dark class="mx-3" @click='crearPartida()'>
              <v-icon>mdi-plus</v-icon>
              Crear
            </v-btn>
          </v-toolbar>
        </template>
        <template v-slot:default="{ items }">
          <v-slide-group multiple show-arrows>
            <v-slide-item v-for="(item, index) in items" :key="item.id">
              <v-btn class="mx-2" depressed rounded @click="seleccionar(item)">
                {{ item.dia }}/{{ item.mes }}/{{ item.año }} ({{index+1}})</v-btn
              >
            </v-slide-item>
          </v-slide-group>
        </template>
      </v-data-iterator>

      <v-card v-if="selected" class="my-5 mx-5" elevation="15">
        <v-form ref="form" eager-validation>
          <v-card-title class="w-100">
            <div class="w-100-2">
              <h3 class="text-h4">
                {{ selected.dia }}/{{ selected.mes }}/{{ selected.año }}
              </h3>
              <v-btn color="deep-purple" @click="calendarDialog = true" icon>
                <v-icon> mdi-calendar </v-icon>
              </v-btn>
            </div>

            <v-btn class="to-right" color="red" icon v-if="!creating" @click="deletePartida()">
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
                  class="align-center"
                  v-for="(movimiento, index) in items"
                  :key="index"
                >
                  <v-col cols="5">
                    <v-select
                      filled
                      label="Cuenta"
                      :rules="rules"
                      :items="cuentas"
                      v-model="movimiento.cuenta"
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
                      @input="checkBalance()"
                      filled
                      type="number"
                      v-model="movimiento.debe"
                      min="0"
                      label="Debe"
                      prefix="$"
                      :disabled="movimiento.haber != 0"
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="3">
                    <v-text-field
                      @input="checkBalance()"
                      filled
                      type="number"
                      v-model="movimiento.haber"
                      min="0"
                      label="Haber"
                      prefix="$"
                      :disabled="movimiento.debe != 0"
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="1">
                    <v-btn icon color="red" @click="deleteMovimiento(index)">
                      <v-icon> mdi-delete </v-icon>
                    </v-btn>
                  </v-col>
                </v-row>
              </template>
            </v-data-iterator>
            <center>
              <v-btn icon color="green" @click="crearMovimiento()">
                <v-icon>mdi-plus-circle</v-icon>
              </v-btn>
            </center>
            <v-row class="mt-3">
              <v-col cols="5">
                <v-textarea
                  label="Descripción"
                  v-model="selected.descripcion"
                  :rules="rules"
                >
                </v-textarea>
              </v-col>
              <v-col cols="3">
                <v-text-field
                  filled
                  readonly
                  v-model="totalDebe"
                  label="Total Debe"
                  :rules="balanced"
                  prefix="$"
                >
                </v-text-field>
              </v-col>
              <v-col cols="3">
                <v-text-field
                  filled
                  readonly
                  v-model="totalHaber"
                  label="Total Debe"
                  :rules="balanced"
                  prefix="$"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-card-text>
          <v-divider></v-divider>
          <v-card-actions>
            <v-btn text color="green" @click="save()">
              <v-icon class="mr-2">mdi-content-save</v-icon>
              Guardar
            </v-btn>
          </v-card-actions>
          <v-dialog v-model="calendarDialog" persistent width="300px" v-if="selected">
            <v-date-picker v-model="date"></v-date-picker>
            <v-btn color="green" dark @click="guardarFecha()">
              Aceptar
            </v-btn>
          </v-dialog>
        </v-form>
      </v-card>
    </v-container>
  </div>
</template>

<script>
import { db } from "../firebase.js";
import firebase from 'firebase/app';
export default {
  name: "Partidas",
  data() {
    return {
      calendarDialog: false,
      date: null,
      descendente: false,
      partidas: [],
      cuentas: [],
      busqueda: "",
      ordenar: "fecha",
      dente: false,
      creating: false,
      selected: null,
      rules: [(v) => !!v || "Debe llenar este campo."],
      balanced: [true],
    };
  },
  methods: {
    seleccionar(item) {
      this.creating = false;
      this.selected = item;
      this.date = "";
    },
    crearPartida(){
      this.creating = true;
      console.log(this.selected)
      this.selected = {
        fecha: new firebase.firestore.Timestamp.fromDate(new Date('01-01-2021')),
        movimientos: [],
        descripcion: '',
        año: 2021,
        dia: 1,
        mes: 1
      }
    },
    deletePartida(){
      console.log(this.selected.id);
      db.collection('partidas').doc(this.selected.id).delete();
      this.selected = null;
    },
    crearMovimiento() {
      console.log("crear movimiento");
      if (!this.selected.movimientos)
        this.selected = {
          ...this.selected,
          id: this.selected.id,
          movimientos: [],
        };

      this.selected.movimientos.push({
        cuenta: null,
        debe: 0,
        haber: 0,
      });

      console.log(this.selected);
    },
    deleteMovimiento(index) {
      this.selected.movimientos.splice(index, 1);
    },
    checkBalance() {
      if (this.totalDebe != this.totalHaber) {
        this.balanced = ["La partida debe estar balanceada."];
      } else {
        this.balanced = [true];
      }
    },
    guardarFecha(){
      console.log(this.date);
      var fecha = new Date(this.date);
      fecha.setDate(fecha.getDate() + 1);
      this.selected.fecha = firebase.firestore.Timestamp.fromDate(fecha);
      this.selected.año = fecha.getFullYear();
      this.selected.mes = fecha.getMonth()+1;
      this.selected.dia = fecha.getDate();
      this.calendarDialog = false;

    },
    save() {
      if (!this.$refs.form.validate()) {
        return;
      }
      console.log(this.selected);

      this.selected.movimientos.forEach((movimiento)=>{
        if(movimiento.debe == null){
          movimiento.debe = 0;  
        }
        if(movimiento.haber == null){
          movimiento.haber = 0;
        }
        movimiento.debe = parseFloat(movimiento.debe);
        movimiento.haber = parseFloat(movimiento.haber);
      });

      var partidaRef;
      if (this.selected.id){
        partidaRef = db.collection("partidas").doc(this.selected.id);
        partidaRef.set(this.selected).then(() => {
          console.log("partida guardada!");
        });
      }else{
        partidaRef = db.collection("partidas").doc();
        partidaRef.set(this.selected).then(() => {
          console.log("partida guardada!");
        });
        this.selected = null;
      }

    },
  },
  computed: {
    totalDebe() {
      var total = 0;
      if (!this.selected.movimientos) {
        return 0;
      }
      this.selected.movimientos.forEach((movimiento) => {
        total += Number(movimiento.debe);
      });
      return total;
    },
    totalHaber() {
      var total = 0;
      if (!this.selected.movimientos) {
        return 0;
      }
      this.selected.movimientos.forEach((movimiento) => {
        total += Number(movimiento.haber);
      });
      return total;
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