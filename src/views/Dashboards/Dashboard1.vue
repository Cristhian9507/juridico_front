<template>
  <b-container fluid>
    <b-row>
      <b-col lg="12">
        <b-row>
          <b-col md="6" lg="3">
            <iq-card class-name="iq-card-block iq-card-stretch iq-card-height" body-class="iq-bg-primary rounded">
              <template v-slot:body>
                <div class="d-flex align-items-center justify-content-between">
                  <div class="rounded-circle iq-card-icon bg-primary"><i class="ri-user-fill"></i></div>
                  <div class="text-right">
                    <h2 class="mb-0"><span class="counter">{{ cantidadUsuarios }}</span></h2>
                    <h5 class="">Usuarios</h5>
                  </div>
                </div>
              </template>
            </iq-card>
          </b-col>
          <b-col md="6" lg="3">
            <iq-card class-name="iq-card-block iq-card-stretch iq-card-height" body-class="iq-bg-warning rounded">
              <template v-slot:body >
                <div class="d-flex align-items-center justify-content-between">
                  <div class="rounded-circle iq-card-icon bg-warning"><i class="ri-time-line"></i></div>
                  <div class="text-right">
                    <h2 class="mb-0"><span class="counter">{{ procesosAbiertos }}</span></h2>
                    <h5 class="">Proc. abiertos</h5>
                  </div>
                </div>
              </template>
            </iq-card>
          </b-col>
          <b-col md="6" lg="3">
            <iq-card class-name="iq-card-block iq-card-stretch iq-card-height" body-class="iq-bg-success rounded">
              <template v-slot:body >
                <div class="d-flex align-items-center justify-content-between">
                  <div class="rounded-circle iq-card-icon bg-success"><i class="ri-checkbox-circle-line"></i></div>
                  <div class="text-right">
                    <h2 class="mb-0"><span class="counter">{{ procesosCerrados }}</span></h2>
                    <h5 class="">Proc. cerrados</h5>
                  </div>
                </div>
              </template>
            </iq-card>
          </b-col>
          <b-col md="6" lg="3">
            <iq-card class-name="iq-card-block iq-card-stretch iq-card-height" body-class="iq-bg-info rounded">
              <template v-slot:body >
                <div class="d-flex align-items-center justify-content-between">
                  <div class="rounded-circle iq-card-icon bg-info"><i class="ri-government-line"></i></div>
                  <div class="text-right">
                    <h2 class="mb-0"><span class="counter">{{ audienciasPendientes }}</span></h2>
                    <h5 class="">Audiencias</h5>
                  </div>
                </div>
              </template>
            </iq-card>
          </b-col>
        </b-row>
      </b-col>
      <b-col sm="12">
        <iq-card class-name="iq-card-block iq-card-stretch iq-card-height">
          <template v-slot:headerTitle>
            <h4 class="card-title">Bienvenidos a Juridico App</h4>
          </template>
          <b-row>
            <b-col lg="6">
              <iq-card>
                <template v-slot:headerTitle>
                  <h4>Ingreso de procesos (activos) en los últimos años 6 años</h4>
                </template>
                <template v-slot:body>
                  <IngresosProcesos ref='chartIngresosProcesos' element="ingresosprocesos"/>
                </template>
              </iq-card>
            </b-col>
            <b-col lg="6">
              <iq-card>
                <template v-slot:headerTitle>
                  <h4>Procesos activos por clínica</h4>
                </template>
                <template v-slot:body>
                  <GraficaProcesosPorClinica ref='chartClinicas' element="patient"/>
                </template>
              </iq-card>
            </b-col>
          </b-row>
          <b-row>
            <b-col lg="6">
              <b-col sm="5" md="7"  class="my-1" >
                <b-form-group
                  label="Clínica"
                  label-cols-sm="3"
                  label-cols-md="3"
                  label-cols-lg="4"
                  label-align-sm="left"
                  label-size="sm"
                  class="mb-0"
                >
                  <v-select
                    multiple
                    v-model="clinicasIds"
                    :options="clinicaOptions"
                    :reduce="label => label.code"
                    label="label"
                    @input="cambioFiltro()"
                    id="clinica_id"
                    >
                    <span slot="no-options">No hay clínicas.</span>
                  </v-select>
                </b-form-group>
              </b-col>
            </b-col>
            <b-col lg="6">
              <b-col sm="5" md="7"  class="my-1" >
                <b-form-group
                  label="Tipo de Proceso"
                  label-cols-sm="3"
                  label-cols-md="3"
                  label-cols-lg="4"
                  label-align-sm="left"
                  label-size="sm"
                  class="mb-0"
                >
                  <v-select
                    v-model="tipoProceso"
                    :options="tipoProcesosOptions"
                    :reduce="label => label.code"
                    label="label"
                    id="tipo_id"
                    @input="cambioFiltro()"
                    >
                    <span slot="no-options">No hay tipos de procesos.</span>
                  </v-select>
                </b-form-group>
              </b-col>
            </b-col>
          </b-row>
          <b-row>
            <b-col lg="12">
              <iq-card>
                <template v-slot:headerTitle>
                  <h4>Nivel de éxito</h4>
                </template>
                <template v-slot:body>
                  <b-table
                    :items="procesosNivelExito"
                    :fields="fields"
                    stacked="md"
                    show-empty
                    small
                    style='overflow: auto; border: 3px solid lightgray;'
                  >
                    <template #cell(name)="row">
                      {{ row.value.first }} {{ row.value.last }}
                    </template>
                  </b-table>
                </template>
              </iq-card>
            </b-col>
          </b-row>
        </iq-card>
      </b-col>
    </b-row>
  </b-container>
</template>
<script>
import { xray } from '../../config/pluginInit'
import axios from 'axios'
import Vue from 'vue'
import auth from '@/logic/auth'

export default {
  name: 'Dashboard1',
  mounted () {
    xray.index()
    this.obtenerNumeroUsuarios()
    this.fetchClinicaOptions()
    setTimeout(() => {
      this.obtenerDatosNivelExito()
      this.obtenerCantidadProcesosAbiertos()
      this.obtenerCantidadProcesosCerrados()
      this.obtenerCantidadAudienciasPendientes()
    }, 500)
  },
  computed: {
    userLogged () {
      if (auth.getUserLogged() !== undefined) {
        return JSON.parse(auth.getUserLogged())
      } else {
        return null
      }
    }
  },
  data: function () {
    return {
      cantidadUsuarios: '',
      procesosAbiertos: '',
      procesosCerrados: '',
      audienciasPendientes: '',
      clinicasInforme: [],
      totalesInformePorClinicas: [],
      categoriasClinicas: [],
      heightGraficas: 250,
      clinicasIds: null,
      clinicaOptions: [],
      tipoProcesosOptions: [{ code: 0, label: 'Todos' }, { code: 1, label: 'Responsabilidad Medica' }, { code: 2, label: 'Laborales' }, { code: 3, label: 'Otros' }],
      tipoProceso: 0,
      ingresoProcesos: {
        type: 'bar',
        bodyData: [],
        xKeys: 'año',
        yKeys: ['total'],
        colors: [ '#36A2EB' ],
        labels: [ 'Procesos' ]
      },
      intentos: 0,
      errores: {},
      procesosNivelExito: [],
      fields: [
        { key: 'prore_sentencia_final', label: 'Clasificación Sentencia', class: 'text-center' },
        {
          key: 'total_pagado_clinica',
          label: 'Total pagado clínica',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        },
        {
          key: 'total_pagado_tercero',
          label: 'Total pagado por tecero',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        },
        {
          key: 'total_sentencia',
          label: 'Total sentencia',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        },
        {
          key: 'cuantia_pretensiones',
          label: 'Cuantía de las pretensiones',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        },
        {
          key: 'estimacion_pago_perju_materiales',
          label: 'Suma de estimación perjuicios Mat.',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        },
        {
          key: 'estimacion_pago_perju_inmateriales',
          label: 'Suma de estimación perjuicios Inmat.',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        },
        {
          key: 'total_estimaciones',
          label: 'Total estimaciones',
          formatter: (value, key, item) => {
            return this.formatPrice(value, key, item)
          },
          class: 'text-center'
        }
      ]
    }
  },
  methods: {
    obtenerNumeroUsuarios: function () {
      axios.get('/users/obtenerCantidadUsuarios').then(res => {
        if (res.data.status_code === 200) {
          this.cantidadUsuarios = res.data.cantidad_usuarios
        } else {
          alert('Datos no validos')
        }
      })
    },
    obtenerCantidadProcesosAbiertos: function () {
      if (this.userLogged.usr_id != null && this.userLogged.usr_id !== '') {
        axios.get('/process/obtenerCantidadProcesosAbiertos/' + this.userLogged.usr_id + '/' + this.clinicasIds + '/' + this.tipoProceso).then(res => {
          if (res.data.status_code === 200) {
            this.procesosAbiertos = res.data.cantidad_procesos_abiertos
          } else {
            alert('Datos no validos')
          }
        })
      } else {
        Vue.swal('Usuario no logueado o inactivo')
      }
    },
    obtenerCantidadProcesosCerrados: function () {
      if (this.userLogged.usr_id != null && this.userLogged.usr_id !== '') {
        axios.get('/process/obtenerCantidadProcesosCerrados/' + this.userLogged.usr_id + '/' + this.clinicasIds + '/' + this.tipoProceso).then(res => {
          if (res.data.status_code === 200) {
            this.procesosCerrados = res.data.cantidad_procesos_cerrados
          } else {
            alert('Datos no validos')
          }
        })
      } else {
        Vue.swal('Usuario no logueado o inactivo')
      }
    },
    obtenerCantidadAudienciasPendientes: function () {
      if (this.userLogged.usr_id != null && this.userLogged.usr_id !== '') {
        axios.get('/agenda/obtenerCantidadAudienciasPendientes').then(res => {
          if (res.data.status_code === 200) {
            this.audienciasPendientes = res.data.cantidad_audiencias_pendientes
          } else {
            alert('Datos no validos')
          }
        })
      } else {
        Vue.swal('Usuario no logueado o inactivo')
      }
    },
    fetchClinicaOptions: function () {
      if (this.userLogged.usr_id != null && this.userLogged.usr_id !== '') {
        axios.get('/clinicas/obtener-clinicas/' + this.userLogged.usr_id).then(response => {
          this.clinicaOptions = response.data.clinicas
          if (this.clinicaOptions[0] !== undefined) {
            this.intentos = 0
            this.errores = {}
            if (this.clinicaOptions.length === 1) {
              this.clinicaId = this.clinicaOptions[0].code
            } else {
              this.clinicaOptions.push({ code: 0, label: 'Todos' })
              this.clinicasIds = 0
            }
          }
        })
          .catch((err) => {
            this.errores = err
            if (this.intentos < 2) {
              this.fetchClinicaOptions()
              this.intentos++
            }
          })
      } else {
        Vue.swal('Usuario no logueado o inactivo')
      }
    },
    obtenerDatosNivelExito () {
      if (this.userLogged.usr_id != null && this.userLogged.usr_id !== '') {
        axios.get('/process/obtener-datos-nivel-exito/' + this.userLogged.usr_id + '/' + this.clinicasIds + '/' + this.tipoProceso).then(res => {
          if (res.data.status_code === 200) {
            this.intentos = 0
            this.errores = {}
            if (typeof res.data.process === 'undefined') {
              Vue.swal(res.data.mensaje)
            } else {
              this.procesosNivelExito = res.data.process
            }
          } else {
            Vue.swal('Ocurrió un error tratando de obtener los datos')
          }
        })
          .catch((err) => {
            this.errores = err
            if (this.intentos < 2) {
              this.obtenerDatosNivelExito()
              this.intentos++
            }
          })
      } else {
        Vue.swal('Usuario no logueado o inactivo')
      }
    },
    formatPrice (value) {
      let val = (value / 1).toFixed(0).replace('.', ',')
      return '$' + val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.')
    },
    cambioFiltro () {
      if (this.clinicasIds.length === 0) {
        this.clinicasIds = 0
      }
      if (this.clinicasIds[0] === 0) {
        this.clinicasIds.shift()
      }
      this.obtenerDatosNivelExito()
    }
  }
}
</script>

<style>
.iq-card-body{
  flex: unset;
}
</style>
