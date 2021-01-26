<template>
  <div class="empleados">
    <v-container class="my-10 mx-5">
      <h1 class="text-h2">Lista de empleados</h1>
      <v-divider></v-divider>
      <v-data-iterator
        :items="empleados"
        :items-per-page="5"
        no-data-text="Aún no hay cuentas creadas"
        :search="busqueda"
        :sort-by="ordenar"
        :sort-desc="descendente"
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
              @click="create"
              dark
              class="mx-3"
            >
              <v-icon>mdi-plus</v-icon>
              Crear
            </v-btn>
            <v-btn
              depressed
              large
              dark
              color="blue"
              @click="totalesDialog = true"
            >
              <v-icon>mdi-finance</v-icon>
              Totales
            </v-btn>
          </v-toolbar>
        </template>

        <template v-slot:default="{ items }">
          <v-row>
            <v-col cols="4" v-for="item in items" :key="item.id">
              <v-card class="mx-auto height-100" max-width="344" elevation="10">
                <v-card-text>
                  <div>{{ item.area }}</div>
                  <p class="display-1 text--primary">{{ item.nombre }}</p>
                  <p>Salario: ${{ item.salario }} /hr</p>
                  <v-divider></v-divider>
                  <p class="mt-2"><strong> Cuenta </strong></p>
                  <v-row>
                    <v-col cols="6"> + Horas ({{ item.horas }}) </v-col>
                    <v-col cols="6"> ${{ item.horas * item.salario }} </v-col>
                  </v-row>
                  <v-row class="green lighten-5">
                    <v-col cols="6" class="green--text">
                      + Horas extra ({{ item.horasExtra }})
                    </v-col>
                    <v-col cols="6" class="green--text">
                      ${{ item.horasExtra * item.salario }}
                    </v-col>
                  </v-row>
                  <v-row class="red lighten-5">
                    <v-col cols="6" class="red--text"> - Retención AFP</v-col>
                    <v-col cols="6" class="red--text">
                      ${{ calcAFP(salarioTotal(item)) }}
                    </v-col>
                  </v-row>
                  <v-row class="red lighten-5">
                    <v-col cols="6" class="red--text"> - Retención ISSS</v-col>
                    <v-col cols="6" class="red--text">
                      ${{ calcISSS(salarioTotal(item)) }}
                    </v-col>
                  </v-row>
                  <v-row
                    class="red lighten-5"
                    v-if="calcISR(salarioTotal(item))"
                  >
                    <v-col cols="6" class="red--text"> - ISR</v-col>
                    <v-col cols="6" class="red--text">
                      ${{ calcISR(salarioTotal(item)) }}
                    </v-col>
                  </v-row>
                  <v-divider></v-divider>
                  <v-row class="mt-1">
                    <v-col cols="6" class=""> Total a pagar</v-col>
                    <v-col cols="6" class="">
                      ${{ totalPago(salarioTotal(item)) }}
                    </v-col>
                  </v-row>
                </v-card-text>
                <v-card-actions>
                  <v-btn
                    text
                    color="red accent-4"
                    @click="borrarEmpleado(item)"
                  >
                    <v-icon class="mr-1">mdi-delete</v-icon>
                    Borrar
                  </v-btn>
                  <v-btn
                    text
                    color="blue accent-4"
                    @click="editarEmpleado(item)"
                  >
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
    <v-dialog v-model="editDialog" width="600">
      <v-card>
        <v-card-title>
          {{ creating ? "Crear" : "Editar" }} empleado.
        </v-card-title>
        <v-divider></v-divider>
        <v-card-text class="mt-3">
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-text-field
              label="Nombre completo"
              :rules="rules"
              filled
              v-model="empleado.nombre"
            >
            </v-text-field>
            <v-row>
              <v-col cols="6">
                <v-select
                  :rules="rules"
                  filled
                  label="Area"
                  :items="areas"
                  v-model="empleado.area"
                >
                </v-select>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  :rules="rules"
                  label="Salario"
                  prefix="$"
                  filled
                  type="number"
                  v-model="empleado.salario"
                  min="0"
                >
                </v-text-field>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="6">
                <v-text-field
                  :rules="rules"
                  label="Horas"
                  suffix="hrs"
                  filled
                  type="number"
                  v-model="empleado.horas"
                  min="0"
                >
                </v-text-field>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  :rules="rules"
                  label="Horas Extra"
                  suffix="hrs"
                  filled
                  type="number"
                  v-model="empleado.horasExtra"
                  min="0"
                >
                </v-text-field>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-btn text color="green" @click="guardar()">
            <v-icon class="mr-2"> mdi-content-save </v-icon>
            {{ creating ? "Crear" : "Guardar" }}
          </v-btn>
          <v-btn text color="red" @click="cancelar()">
            <v-icon class="mr-2"> mdi-cancel</v-icon>
            Cancelar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="totalesDialog" width="600">
      <v-card>
        <v-card-title> Totales </v-card-title>
        <v-divider></v-divider>
        <v-card-text class="mt-5">
          <v-row>
           <v-col cols="6">Total de salarios.</v-col>
           <v-col cols="6">${{totalVentas + totalAdmin}}</v-col>

           <v-col cols="6">Total por empleados de ventas.</v-col>
           <v-col cols="6">${{totalVentas}}</v-col>

          <v-col cols="6">Total por empleados de administración.</v-col>
          <v-col cols="6">${{totalAdmin}}</v-col>

          <v-col cols="6">Total en retenciones.</v-col>
          <v-col cols="6">${{totalRetenciones}}</v-col>

          <v-col cols="6">Total aportes.</v-col>
          <v-col cols="6">${{aportesVentas + aportesAdmin}}</v-col>

          <v-col cols="6">Aportes patronales en ventas.</v-col>
          <v-col cols="6">${{aportesVentas}}</v-col>

          <v-col cols="6">Aportes patronales en administración.</v-col>
          <v-col cols="6">${{aportesAdmin}}</v-col>

          </v-row>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import { db } from "../firebase.js";
export default {
  name: "Empleados",
  data() {
    return {
      totalesDialog: false,
      empleados: [],
      creating: false,
      valid: false,
      editDialog: false,
      empleado: {
        nombre: "",
        area: "",
        salario: null,
        horas: null,
        horasExtra: null,
      },
      busqueda: "",
      ordenar: null,
      descendente: false,
      areas: ["Ventas", "Administración"],
      sortBy: [
        { text: "Salario", param: "salario" },
        { text: "Nombre", param: "nombre" },
        { text: "Horas", param: "horas" },
        { text: "Horas Extra", param: "horasExtra" },
      ],
      rules: [(v) => !!v || "Debe completar este campo"],
    };
  },
  methods: {
    create() {
      this.empleado = {
        nombre: "",
        area: "",
        salario: null,
        horas: null,
        horasExtra: null,
      };
      this.editDialog = true;
      this.creating = true;
    },
    cancelar() {
      this.empleado = {
        nombre: "",
        area: "",
        salario: null,
        horas: null,
        horasExtra: null,
      };
      this.$refs.form.resetValidation();
      this.$refs.form.reset();
      this.editDialog = false;
    },
    editarEmpleado(empleado) {
      this.empleado = empleado;
      this.editDialog = true;
    },
    guardar() {
      if (!this.$refs.form.validate()) {
        return;
      }

      var docRef;
      this.empleado.salario = parseFloat(this.empleado.salario);
      this.empleado.horas = parseFloat(this.empleado.horas);
      this.empleado.horasExtra = parseFloat(this.empleado.horasExtra);

      if (this.empleado.id) {
        docRef = db.collection("empleados").doc(this.empleado.id);
      } else {
        docRef = db.collection("empleados").doc();
      }
      var context = this;
      docRef.set(this.empleado).then(() => {
        context.editDialog = false;
      });
    },
    salarioTotal(empleado) {
      console.log("Salario: " + empleado.salario);
      return empleado.salario * (empleado.horas + empleado.horasExtra);
    },
    calcAFP(salario) {
      var afp = 0.0725 * Math.min(salario, 6500);
      return Math.round(afp * 100) / 100;
    },
    calcISSS(salario) {
      var isss = 0.03 * Math.min(salario, 1000);
      return Math.round(isss * 100) / 100;
    },
    calcISR(salario) {
      var grabado = salario - this.calcAFP(salario) - this.calcISSS(salario);
      console.log(grabado);
      var isr;

      if (grabado <= 472) {
        isr = 0;
      } else if (grabado <= 895.24) {
        isr = (grabado - 472) * 0.1 + 17.67;
      } else if (grabado <= 2038.1) {
        isr = (grabado - 895.24) * 0.2 + 60;
      } else {
        isr = (grabado - 2038.1) * 0.2 + 288.57;
      }

      return Math.round(isr * 100) / 100;
    },
    totalPago(salario) {
      var total =
        salario -
        this.calcISSS(salario) -
        this.calcAFP(salario) -
        this.calcISR(salario);

      return Math.round(total * 100) / 100;
    },
    borrarEmpleado(empleado) {
      db.collection("empleados")
        .doc(empleado.id)
        .delete()
        .then(() => {
          console.log("Empleado borrado");
        });
    },
  },
  computed:{
    totalVentas(){
      var total = 0;
      var context = this;
      this.empleados.forEach((empleado)=>{
        if(empleado.area == 'Ventas')
          total += context.salarioTotal(empleado);
      });
      return Math.floor(total*100)/100;
    },
    totalAdmin(){
      var total = 0;
      var context = this;
      this.empleados.forEach((empleado)=>{
        if(empleado.area == 'Administración')
          total += context.salarioTotal(empleado);
      });
      return Math.round(total*100)/100;
    },
    totalPagar(){
      var total = this.totalPagarVentas + this.totalPagarAdmin;
      return Math.round(total*100)/100;
    },
    totalRetenciones(){
      var total = 0;
      var con = this;
      this.empleados.forEach((empleado)=>{
        total += con.calcAFP(con.salarioTotal(empleado));
        total += con.calcISSS(con.salarioTotal(empleado));
        total += con.calcISR(con.salarioTotal(empleado));
      });

      return Math.round(total*100)/100;
    },
    aportesVentas(){
      var total = 0;
      var con = this;

      this.empleados.forEach((empleado)=>{
        if(empleado.area == "Ventas"){
          total += con.salarioTotal(empleado)*0.075;
          total += con.salarioTotal(empleado)*0.075;
        }
      });

      return Math.round(total*100)/100;
    },
    aportesAdmin(){
      var total = 0;
      var con = this;

      this.empleados.forEach((empleado)=>{
        if(empleado.area == "Administración"){
          total += con.salarioTotal(empleado)*0.075;
          total += con.salarioTotal(empleado)*0.075;
        }
      });

      return Math.round(total*100)/100;
    }
  },
  firestore: {
    empleados: db.collection("empleados"),
  },
};
</script>

<style scoped>
.height-100{
  min-height:100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
</style>