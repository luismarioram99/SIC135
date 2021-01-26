<template>
  <div class="balance">
    <v-container class="my-10 mx-5">
      <h1 class="text-h2">Balance de comprobación</h1>
      <v-divider></v-divider>
    </v-container>

    <v-card class="mx-auto my-5 height-100" max-width="344" elevation="10">
      <v-card-text>
        <p class="display-1 text--primary">Consolidación</p>
        
        <v-row >
          <v-col cols="6" class="">
            Total debe: 
          </v-col>
          <v-col cols="6" >
            $ {{totalDebe}}
          </v-col>
        </v-row>
        <v-row >
          <v-col cols="6" class="">
            - Total haber: 
          </v-col>
          <v-col cols="6">
            $ {{totalHaber}}
          </v-col>
        </v-row>
        <v-divider></v-divider>

        <v-row class="mt-2 green lighten-4">
          <v-col cols="6" class="green--text">
            <strong>= Diferencia </strong>
          </v-col>
          <v-col cols="6" >
            <strong>

            $ {{totalDebe - totalHaber}}
            </strong>
          </v-col>
        </v-row>
        
      </v-card-text>
    </v-card>

    <v-card class="mx-5 my-5" elevation="15">
      <v-data-table
        hide-default-footer
        :items="cuentasFiltrado"
        :headers="headers"
        items-per-page="-1"
      >
        <template v-slot:[`item.debe`]="{ item }">
          {{ cuentas[item.numero].debe?"$ " +cuentas[item.numero].debe : "-" }}
        </template>
        <template v-slot:[`item.haber`]="{ item }">
          {{ cuentas[item.numero].haber?"$ " + cuentas[item.numero].haber : "-" }}
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script>
import { db } from "../firebase.js";
export default {
  name: "Balance",
  data() {
    return {
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
      headers: [
        { text: "Núm", value: "numero" },
        { text: "Cuenta", value: "nombre" },
        { text: "Debe", value: "debe" },
        { text: "Haber", value: "haber" },
      ],
      partidas: [],
    };
  },
  computed: {
    cuentas() {
      var cuentas = {};

      this.partidas.forEach((partida) => {
        partida.movimientos.forEach((movimiento) => {
          if (!cuentas[movimiento.cuenta.categoria.numero]) {
            console.log(movimiento.cuenta.categoria.numero);
            console.log(movimiento.debe, movimiento.haber);
            cuentas[movimiento.cuenta.categoria.numero] = {};
            cuentas[movimiento.cuenta.categoria.numero].debe = 0;
            cuentas[movimiento.cuenta.categoria.numero].haber = 0;
          }

          cuentas[movimiento.cuenta.categoria.numero].debe += parseFloat(
            movimiento.debe
          );
          cuentas[movimiento.cuenta.categoria.numero].haber += parseFloat(
            movimiento.haber
          );
        });
      });

      for (var cuenta in cuentas) {
        var nuevo_debe, nuevo_haber;
        nuevo_debe = Math.max(cuentas[cuenta].debe - cuentas[cuenta].haber, 0);
        nuevo_haber = Math.max(cuentas[cuenta].haber - cuentas[cuenta].debe, 0);
        cuentas[cuenta].debe = nuevo_debe;
        cuentas[cuenta].haber = nuevo_haber;
      }

      return cuentas;
    },
    cuentasFiltrado() {
      var filtrados = this.categorias.filter((categoria) => {
        if (!this.cuentas[categoria.numero]) return false;
        if (
          !this.cuentas[categoria.numero].debe &&
          !this.cuentas[categoria.numero].haber
        )
          return false;
        return true;
      });
      return filtrados;
    },
    totalDebe() {
      var total = 0;

      this.cuentasFiltrado.forEach((cuenta) => {
        total += this.cuentas[cuenta.numero].debe;
      });

      return total;
    },
    totalHaber() {
      var total = 0;

      this.cuentasFiltrado.forEach((cuenta) => {
        total += this.cuentas[cuenta.numero].haber;
      });

      return total;
    },
  },
  firestore: {
    partidas: db.collection("partidas"),
  },
};
</script>

<style scoped>
</style>