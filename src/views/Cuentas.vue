<template>
  <div class="cuentas">
    <v-container class="my-10 mx-5">
      <h1 class="text-h2">Catálogo de cuentas</h1>

      <v-divider></v-divider>

      <v-data-iterator
        class="my-5"
        :items="cuentas"
        :items-per-page="5"
        no-data-text="Aún no hay cuentas creadas"
        :search="busqueda"
      >
        <template v-slot:header>
          <v-toolbar class="mb-5 mt-5" elevation="0">
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
              :items="params"
              item-text="texto"
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
              @click="dialog = true"
              dark
              class="mx-3"
            >
              <v-icon>mdi-plus</v-icon>
              Crear
            </v-btn>
          </v-toolbar>
        </template>

        <template v-slot:default="{ items }">
          <v-row>
            <v-col cols="4" v-for="item in items" :key="item.numero">
              <v-card class="mx-auto" max-width="344" elevation="10">
                <v-card-text>
                  <div>{{ item.numero }}</div>
                  <p class="display-1 text--primary">{{ item.nombre }}</p>
                  <p>
                    {{ item.categoria.numero }} - {{ item.categoria.nombre }}
                  </p>
                  <div class="text--primary">
                    {{ item.descripcion }}
                  </div>
                </v-card-text>
                <v-card-actions>
                  <v-btn text color="red accent-4" @click="borrar(item)">
                    <v-icon class="mr-1">mdi-delete</v-icon>
                    Borrar
                  </v-btn>
                  <v-btn text color="blue accent-4" @click="editar(item)">
                    <v-icon class="mr-1">mdi-file-edit</v-icon>
                    Editar
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </template>
      </v-data-iterator>
    </v-container>

    <v-dialog v-model="dialog" max-width="800">
      <v-card>
        <v-card-title>
          <span class="headline">Nuevo Catalogo</span>
        </v-card-title>
        <v-card-text>
          <v-container fluid>
            <v-form ref="form" v-model="valid" lazy-validation>
              <v-row>
                <v-col>
                  <v-text-field
                    :rules="rules"
                    filled
                    label="Número"
                    hint="número de cuenta"
                    placeholder="######"
                    v-model="nuevaCuenta.numero"
                  >
                  </v-text-field>
                </v-col>
                <v-col>
                  <v-select
                    :rules="rules"
                    :items="categorias"
                    v-model="nuevaCuenta.categoria"
                    label="Categoría"
                    filled
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
              </v-row>
              <v-text-field
                :rules="rules"
                filled
                label="Nombre"
                hint="Nombre de la cuenta"
                v-model="nuevaCuenta.nombre"
              >
              </v-text-field>
              <v-textarea
                :rules="rules"
                filled
                label="Descripción"
                v-model="nuevaCuenta.descripcion"
              >
              </v-textarea>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" dark @click="cancelar()">
            <v-icon class="mr-1"> mdi-cancel </v-icon>
            Cancelar
          </v-btn>
          <v-btn color="blue darken-1" dark @click="agregar()">
            <v-icon class="mr-1"> mdi-plus </v-icon>
            Crear
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="editarDialog" max-width="800">
      <v-card>
        <v-card-title>
          <span class="headline">Editar Catalogo</span>
        </v-card-title>
        <v-card-text>
          <v-container fluid>
            <v-form ref="editForm" v-model="valid" lazy-validation>
              <v-row>
                <v-col>
                  <v-text-field
                    :rules="rules"
                    filled
                    label="Número"
                    hint="número de cuenta"
                    placeholder="######"
                    v-model="editarCuenta.numero"
                  >
                  </v-text-field>
                </v-col>
                <v-col>
                  <v-select
                    :rules="rules"
                    :items="categorias"
                    v-model="editarCuenta.categoria"
                    label="Categoría"
                    filled
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
              </v-row>
              <v-text-field
                :rules="rules"
                filled
                label="Nombre"
                hint="Nombre de la cuenta"
                v-model="editarCuenta.nombre"
              >
              </v-text-field>
              <v-textarea
                :rules="rules"
                filled
                label="Descripción"
                v-model="editarCuenta.descripcion"
              >
              </v-textarea>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" dark @click="cancelar()">
            <v-icon class="mr-1"> mdi-cancel </v-icon>
            Cancelar
          </v-btn>
          <v-btn color="blue darken-1" dark @click="guardarEditado()">
            <v-icon class="mr-1"> mdi-content-save </v-icon>
            Guardar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { db } from "../firebase.js";
export default {
  name: "Cuentas",
  data() {
    return {
      cuentas: [],
      busqueda: "",
      params: [],
      ordenar: "",
      descendente: false,
      nuevaCuenta: {},
      editarCuenta: {},
      dialog: false,
      valid: true,
      editarDialog: false,
      categorias: [
        { numero: "1101", nombre: "Efectivo y Equivalentes" },
        { numero: "1102", nombre: "Inversiones Financieras a Corto Plazo" },
        { numero: "1103", nombre: "Cuentas Por Cobrar" },
        { numero: "1104", nombre: "IVA Crédito Fiscal" },
        {
          numero: "1105",
          nombre: "Cuentas Por Cobrar Arrendamiento Financiero",
        },
        { numero: "1106", nombre: "Partes Relacionadas Corto Plazo" },
        { numero: "1107", nombre: "Inventarios" },
        { numero: "1108", nombre: "Inversiones Temporales" },
        { numero: "1109", nombre: "Gastos Pagados por Anticipado" },
        { numero: "1201", nombre: "Propiedades, Planta y Equipo" },
        { numero: "1202", nombre: "Inversiones Permanentes" },
        { numero: "1203", nombre: "Impuesto Diferido ISR- Activo" },
        { numero: "1204", nombre: "Activos Intangibles" },
        { numero: "1205", nombre: "Cuentas por Cobrar a Largo Plazo" },
        { numero: "1206", nombre: "Otras Cuentas por Cobrar – Largo Plazo" },
        { numero: "1207", nombre: "Depósitos en Garantía – Largo Plazo" },
        {
          numero: "1208",
          nombre: "Cuentas por Cobrar Arrendamiento Financiero – Largo Plazo",
        },
        { numero: "1209", nombre: "Partes Relacionadas Largo Plazo" },
        {
          numero: "2101",
          nombre: "Préstamos y Sobregiros Bancarios a Corto Plazo",
        },
        { numero: "2102", nombre: "Proveedores y Otras Cuentas Por Pagar" },
        {
          numero: "2103",
          nombre: "Obligaciones Bajo Arrendamiento Financiero",
        },
        {
          numero: "2104",
          nombre: "Otros Acreedores, Retenciones y Descuentos",
        },
        { numero: "2105", nombre: "Beneficios a Empleados Por Pagar" },
        { numero: "2106", nombre: "IVA Débito Fiscal" },
        { numero: "2107", nombre: "Impuesto Sobre La Renta Por Pagar" },
        { numero: "2108", nombre: "Dividendos Por Pagar" },
        { numero: "2109", nombre: "Partes Relacionadas Corto Plazo" },
        { numero: "2201", nombre: "Préstamos Bancarios a Largo Plazo" },
        {
          numero: "2202",
          nombre: "Obligaciones Bajo Arrendamiento Financiero – Largo Plazo",
        },
        { numero: "2203", nombre: "Anticipos y Garantías de Clientes" },
        {
          numero: "2204",
          nombre: "Beneficios a Empleados Por Pagar a Largo Plazo",
        },
        { numero: "2205", nombre: "Partes Relacionadas Largo Plazo" },
        { numero: "3101", nombre: "Capital Social" },
        { numero: "3201", nombre: "Utilidades Restringidas" },
        { numero: "3202", nombre: "Utilidades No Distribuidas" },
        { numero: "3203R", nombre: "Pérdidas Acumuladas" },
        {
          numero: "3204",
          nombre: "Superávit Por Revaluación en Propiedades, Planta Y Equipo.",
        },
        {
          numero: "3205",
          nombre: "Efectos por Adopción de NIIF para las Pymes",
        },
        { numero: "4101", nombre: "Costo de Venta" },
        { numero: "4102", nombre: "Costo de Producción" },
        { numero: "4103", nombre: "Gastos de Venta" },
        { numero: "4104", nombre: "Gastos de Administración" },
        { numero: "4201", nombre: "Gastos Financieros" },
        {
          numero: "4202",
          nombre:
            "Gastos por Cambios en valor Razonable en Inversiones Financieras",
        },
        { numero: "4203", nombre: "Otros Gastos de no Operación" },
        {
          numero: "4301",
          nombre: "Gastos de Impuesto Sobre la Renta Corriente",
        },
        {
          numero: "4302",
          nombre: "Gastos de Impuesto Sobre la Renta Diferido - Activo",
        },
        {
          numero: "4303",
          nombre: "Gastos de Impuesto Sobre la Renta Diferido – Pasivo",
        },
        { numero: "5101", nombre: "Ingresos por Ventas" },
        { numero: "5102", nombre: "Otros Ingresos de Operación" },
        { numero: "5201", nombre: "Ingresos Financieros" },
        {
          numero: "5202",
          nombre:
            "Ingresos por cambios en Valor Razonable de Inversiones Financieras",
        },
        {
          numero: "5203",
          nombre:
            "Reversión del Deterioro y Ganancia en los Instrumentos Financieros",
        },
        { numero: "5204", nombre: "Otros Ingresos de no Operación" },
        { numero: "6101", nombre: "Pérdidas y Ganancias" },
      ],
      rules: [(v) => !!v || "Debe llenar este campo."],
    };
  },
  firestore: {
    cuentas: db.collection("cuentas"),
  },
  methods: {
    agregar() {
      if (this.$refs.form.validate()) {
        console.log(this.nuevaCuenta);
        var context = this;
        db.collection("cuentas")
          .add(this.nuevaCuenta)
          .then(() => {
            context.$refs.form.reset();
            context.dialog = false;
          });
      }
    },
    cancelar() {
      this.$refs.form.reset();
      this.dialog = false;
    },
    borrar(item) {
      db.collection("cuentas").doc(item.id).delete();
    },
    editar(item){
      this.editarCuenta = item;
      this.editarDialog = true;
    },
    guardarEditado(){
      if(this.$refs.editForm.validate()){
        var context = this;
        db.collection('cuentas').doc(this.editarCuenta.id).set({
          numero: this.editarCuenta.numero,
          nombre: this.editarCuenta.nombre,
          categoria: this.editarCuenta.categoria,
          descripcion: this.editarCuenta.descripcion          
        }).then(()=>{
          context.$refs.editForm.reset();
          context.editarDialog = false;
        })

      }
    }
  },
};
</script>

<style scoped>
</style>