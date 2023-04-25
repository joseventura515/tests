<template>
    <div>
        <!-- INICIO MODAL RANGOS CUOTAS-->
        <ods-dialog
        :visible.sync="modalRangosCuota.dialogVisible"
        :width="modalRangosCuota.width"
        @close="handleCloseModal"
        :close-on-click-modal="false"
        :close-on-press-escape="false"
        :title="tableTitle">
        <div>
            <!-- layout with 24 columns, to display 6 tables so 4 per table -->
            <ods-row>
                <ods-table
                :data="dataTableModalRangosCuotaForPage"
                stripe
                style="width: 100%; overflow:auto"
                header-row-class-name="headerTableStyle"
                :default-sort = "{'prop': 'numAlfanumerico', 'order': 'ascending'}"
                highlight-current-row
                @current-change="handleCurrentChangeCuotasRangos"
                @sort-change="sortChanged"
                @cell-dblclick="dblClickEditarCuotasRangos()"
                height="30vh"
                border
                size="mini">
                    <!-- NUM ALFA -->
                    <ods-table-column
                        prop="numAlfanumerico"
                        sortable
                        align="left"
                        header-align="left"
                        :label="$t('remitentes.numAlfa')"
                        min-width="100">
                    </ods-table-column>
                    <!-- FECHA ALTA -->
                    <ods-table-column
                        prop="fechaAlta"
                        sortable
                        align="left"
                        header-align="left"
                        :label="$t('remitentes.fechaAlta')"
                        min-width="100">
                    </ods-table-column>
                    <!-- FECHA BAJA -->
                    <ods-table-column
                        prop="fechaBaja"
                        sortable
                        align="left"
                        header-align="left"
                        :label="table3Headers"
                        min-width="100">
                    </ods-table-column>
                    <!-- ACCIONES -->
                    <ods-table-column
                        prop="accion"
                        align="center"
                        header-align="left"
                        :label="$t('acciones')"
                        width="100">
                    <template slot-scope="scope">
                        <a @click="editarRegistroCuotasRangos(scope.row)"><ods-icon class="ods-icon-pencil" style="margin-right: 5px;" size="20" title="Editar"/></a>
                        <a @click="eliminarRegistroCuotasRangos(scope.row)"><ods-icon class="ods-icon-trash-o" size="20" title="Baja Lógica"/></a>
                    </template>
                    </ods-table-column>
                </ods-table>
            </ods-row>
            <ods-row class="ods-button-row">
                <ods-col  :span="24" :md="12">
                    <ods-pagination class="align-right"
                        @current-change="handleChangesPageModalRangosCuotas"
                        @size-change="handleSizeChangeModalRangosCuotas"
                        :current-page.sync="modalRangosCuota.currentPage"
                        :page-size="modalRangosCuota.numRegisters"
                        :pageSizes="[2,5]"
                        layout="total, prev, pager, next, jumper"
                        :total="modalRangosCuota.totalItems">
                    </ods-pagination>
                </ods-col>
                <ods-col :span="24" :md="12" class="optionsButtons" >
                    <ods-button @click="newRangosCuotas()">{{ $t('button.nuevo') }}</ods-button>
                </ods-col>
            </ods-row>
            <ods-row v-if="modalRangosCuota.showForm">
                <ods-form ref="formTableAlfanumerico" :rules="rulesRangosCuotas" :model="formTableAlfanumerico" label-position="top">
                    <ods-row>
                    <ods-col :md="6" class="filters">
                        <ods-form-item :label="$t('remitentes.alfanumerico')" prop="numAlfanumerico">
                            <ods-input :disabled="modalRangosCuota.inputDisabled"
                                :maxlength="20"
                                :minlength="0"
                                v-model="formTableAlfanumerico.numAlfanumerico"
                                :placeholder="$t('remitentes.alfanumerico')">
                            </ods-input>
                        </ods-form-item>
                    </ods-col>
                    <ods-col :md="7" class="filters">
                        <ods-form-item prop="fechaAlta" :label="$t('fechaAlta')">
                            <ods-date-picker :placeholder="$t('fechaAlta')"
                                :picker-options="datepickerOptions"
                                v-model="formTableAlfanumerico.fechaAlta"
                                style="width:98%" maxlength="10"
                                format="dd/MM/yyyy"
                                value-format="dd/MM/yyyy"/>
                        </ods-form-item>
                    </ods-col>
                    <ods-col :md="7" class="filters">
                        <ods-form-item prop="fechaBaja" :label="$t('fechaBaja')">
                            <ods-date-picker :placeholder="$t('fechaBaja')"
                                :picker-options="datepickerOptions"
                                v-model="formTableAlfanumerico.fechaBaja"
                                style="width:98%" maxlength="10"
                                format="dd/MM/yyyy"
                                value-format="dd/MM/yyyy"/>
                        </ods-form-item>
                    </ods-col>
                    <ods-col :md="2">
                        <ods-button class="component-vertical-center" @click="saveAddRangosCuotas()">Ok</ods-button>
                    </ods-col>
                    </ods-row>
                </ods-form>
            </ods-row>
        </div>
        <div slot="footer" class="dialog-footer text-center">
            <ods-button @click="saveAllRangosCuotas()">{{$t('guardar')}}</ods-button>
            <ods-button type="neutral" @click="cancelarModalRangosCuotas">{{$t('cancelar')}}</ods-button>
        </div>
        </ods-dialog>
        <!-- FIN MODAL RANGOS CUOTAS-->
    </div>
</template>
<script>
/* eslint-disable */
import myProperties from '../../../static/properties.json'
import {getSysDate, formatDateToFormat } from '../../utils/pam-util'
import {checkIfAlfanumericoExistsAsRemitente, checkIfAlfanumericoExists, getRemitenteDataAlfanumericos} from '../../utils/pam-api'
import { Alert } from 'element-ui'


export default {
    name: 'ComponentModalCaudalCuota',
    props: {
        visibility: {
            type: Boolean
        },
        typeApi: {
            type: String
        },
        tableTitle: {
            type: String
        },
        table3Headers: {
            type: String
        },
        contratoEditCod: {
            type: String
        },
        codNumalfanum: {
            type: String
        },
        fechaAltaRemitente: {
            type: String
        },
        fechaBajaRemitente: {
            type: String
        }
    },
    computed: {
    },
    data () {
        return {
            // MODA CUOTAS
            formIsEdit: false,
            loadCaudalCuotasFirstTime: true,
            formTableAlfanumerico:{
                numAlfanumerico: '',
                fechaAlta: '',
                fechaBaja: '',
                numAlfanumericoSelected: '',
                fechaAltaSelected: '',
                fechaBajaSelected: ''
            },
            datepickerOptions: {
                'disabledDate': function disabledDate(time){return time.getTime()>Date.now()},
            },
            dataTableModalRangosCuota: [],
            dataTableModalRangosCuotaOriginal: [],
            dataTableModalRangosCuotaForPage: [],
            InitialdataTableModalRangosCuota: [],
            modalRangosCuota: {
                showForm: false,
                inputDisabled: false,
                title: 'Caudal Garantizado Mensajes/Seg',
                width: '600px',
                dialogVisible: false, 

                // TABLE INDEX
                currentSortProp: undefined,
                currentSortOrder: undefined,
                totalItems: 0,
                numRegisters: 5,//myProperties.NUM_REG_POR_PAGINA,
                currentPage: 1,
            },
            rulesRangosCuotas: {
                'numAlfanumerico': [
                    {
                        required: true, message: this.$t('message.campoRequerido'), trigger: 'blur'
                    },
                    {
                        // validator: isNumerico, trigger: 'blur'
                    }
                ],
                'fechaAlta': [
                    {
                        required: true, message: this.$t('message.campoRequerido'), trigger: 'blur'
                    },
                    {
                        // validator: isNumerico, trigger: 'blur'
                    }
                ],
                'fechaBaja': [
                    {
                        // required: true, message: this.$t('message.campoRequerido'), trigger: 'blur'
                    },
                    {
                        // validator: isDecimal8and4, trigger: 'blur'
                    }
                ],
            },
            // FIN MODAL CUOTAS
        }
    },    
    methods: {
        // INICIO RANGO CUOTAS
        handleCloseModal () {
            this.$emit('updateModalState', false)
            this.modalRangosCuota.showForm = false
            if(JSON.stringify(this.dataTableModalRangosCuota) === JSON.stringify(this.dataTableModalRangosCuotaOriginal)
                // || JSON.stringify(this.dataTableModalRangosCuota) === JSON.stringify(this.InitialdataTableModalRangosCuota)
            ){
                console.log("NO CAMBIOS")
                this.$emit('handleSaveAllTable', [])
            } else {
                this.dataTableModalRangosCuota = this.InitialdataTableModalRangosCuota
                this.getListaRangosCuotasFiltrado()
            }
        },
        cancelarModalRangosCuotas () {
            this.$confirm(this.$t('contratos.dialogTable.cancelModal'), this.$t('message.title.confirmacion'), {
            cancelButtonText: this.$t('button.cancelar'),
            confirmButtonText: this.$t('button.aceptar'),
            type: 'warning'
            }).then(() => {
                this.dataTableModalRangosCuota = this.InitialdataTableModalRangosCuota
                this.getListaRangosCuotasFiltrado()
                this.$emit('updateModalState', false)
            }).catch(() => {})
        },
        saveAllRangosCuotas () {
                if(JSON.stringify(this.dataTableModalRangosCuota) === JSON.stringify(this.dataTableModalRangosCuotaOriginal)
                    // || JSON.stringify(this.dataTableModalRangosCuota) === JSON.stringify(this.InitialdataTableModalRangosCuota)
                    ){
                    console.log("NO CAMBIOS")
                    this.$emit('handleSaveAllTable', [])
                } else {
                    console.log("SI CAMBIOS")
                    this.$emit('handleSaveAllTable', this.dataTableModalRangosCuota)
                    //ENVIAMOS EL TAMAÑO DE LA LISTA PARA VALIDAR LOS ALFANUMERICOS
                    if (this.dataTableModalRangosCuota.length > 0) {
                        this.$emit('tableModalSize', this.dataTableModalRangosCuota.length)
                    }
                }
                this.$emit('updateModalState', false)
                this.InitialdataTableModalRangosCuota = this.dataTableModalRangosCuota
        },
        saveAddRangosCuotas () {
            this.$refs['formTableAlfanumerico'].validate((valid) => {
                if (valid) {
                    // VALIDAR QUE EL FECHA ALTA < FECHA BAJA
                    var dAlta = new Date(this.formTableAlfanumerico.fechaAlta.split('/')[1]+'/'+this.formTableAlfanumerico.fechaAlta.split('/')[0]+'/'+this.formTableAlfanumerico.fechaAlta.split('/')[2])
                    if (this.formTableAlfanumerico.fechaBaja !== '' && this.formTableAlfanumerico.fechaBaja !== null) {
                        var dBaja = new Date(this.formTableAlfanumerico.fechaBaja.split('/')[1]+'/'+this.formTableAlfanumerico.fechaBaja.split('/')[0]+'/'+this.formTableAlfanumerico.fechaBaja.split('/')[2])
                        var dHoy = getSysDate()
                        if (dAlta.getTime() > dBaja.getTime()) {
                            this.$notify.error({title: this.$t('remitentes.dialogTable.fecAltaMenorFecBaja'), customClass: 'notificacionError'})
                            return false
                        } 
                        // VALIDAR QUE FECHA BAJA > SYSDATE
                        else if (dBaja.getTime() > dHoy.getTime()) {
                            this.$notify.error({title: this.$t('remitentes.dialogTable.fecAltaMenorSysdate'), customClass: 'notificacionError'})
                            return false
                        }
                        // VALIDAR QUE LA FECHA BAJA DE ALFANUM <= FECHA BAJA REMITENTE - SOLO SI FECHA BAJA REM INFORMADA
                        if (this.fechaBajaRemitente !== '') {
                            var dBajaRemitente = new Date(this.fechaBajaRemitente.split('/')[1]+'/'+this.fechaBajaRemitente.split('/')[0]+'/'+this.fechaBajaRemitente.split('/')[2])
                            if (dBajaRemitente.getTime() < dBaja.getTime()) {
                                this.$notify.error({title: this.$t('remitentes.dialogTable.fecBajaMayorAltaRem'), customClass: 'notificacionError'})
                                return false
                            }
                        }
                    }
                    // VALIDAR QUE LA FECHA ALTA DE ALFANUM => FECHA ALTA REMITENTE 
                    var dAltaRemitente = new Date(this.fechaAltaRemitente.split('/')[1]+'/'+this.fechaAltaRemitente.split('/')[0]+'/'+this.fechaAltaRemitente.split('/')[2])
                    if (dAltaRemitente.getTime() > dAlta.getTime()) {
                        this.$notify.error({title: this.$t('remitentes.dialogTable.fecAltaMenorAltaRem'), customClass: 'notificacionError'})
                        return false
                    } 

                    // VALIDAR QUE NO TENGA ESPACIOS EL ALFANUMERICO
                    if (this.formTableAlfanumerico.numAlfanumerico.indexOf(' ') >= 0) {
                        this.$notify.error({title: this.$t('remitentes.dialogTable.alfanumericoNoValido'), customClass: 'notificacionError'})
                        return false
                    }
                    // VERIFICAR QUE EL ALFANUMERICO NO ESTE EN LOS ALFANUMERICOS ONLY IF ALTA
                    var exists = false
                    if (!this.formIsEdit) {
                        this.dataTableModalRangosCuota.forEach((value, index) => {
                            if (value.numAlfanumerico.toString() === this.formTableAlfanumerico.numAlfanumerico.toString()) {
                                exists = true
                            }
                        })
                    }
                    if (exists) {
                        this.$notify.error({title: this.$t('remitentes.dialogTable.AlfaExistsInAlfa'), customClass: 'notificacionError'})
                        return false
                    }
                    // VALIDAR SI EL ALFANBUMERICO YA EXISTE EN PAM_ALFANUMERICOS -> ELIMINADO -1
                    checkIfAlfanumericoExists(this.formTableAlfanumerico.numAlfanumerico).then((resp) => {
                        if (resp.data == -1) {
                            this.$notify.error({title: this.$t('remitentes.dialogTable.alfanumericoExists'), customClass: 'notificacionError'})
                        } else {
                            // VALIDAR SI EXISTE EL ALFANUMERICO COMO REMITENTE
                            checkIfAlfanumericoExistsAsRemitente(this.formTableAlfanumerico.numAlfanumerico,
                                this.formTableAlfanumerico.fechaAlta).then((resp) => {
                                    if (resp.data > 0) {
                                        this.$notify.error({title: this.$t('remitentes.dialogTable.alfanumericoExistsInRemitente').replace('$PARAM1',this.formTableAlfanumerico.numAlfanumerico), customClass: 'notificacionError'})
                                    } else {
                                        //SI ES EDITAR ELIMINAMOS ESE REGISTRO E INSERTAMOS EL NUEVO
                                        var desdeF = this.formTableAlfanumerico.numAlfanumericoSelected
                                        var hastaF = this.formTableAlfanumerico.fechaAltaSelected
                                        var cuotaF = this.formTableAlfanumerico.fechaBajaSelected
                                        var listaFiltrada = this.dataTableModalRangosCuota.filter( function(item) {
                                            return item.numAlfanumerico != desdeF || item.fechaAlta != hastaF || item.fechaBaja != cuotaF
                                        })
                                        this.dataTableModalRangosCuota = listaFiltrada
                                        
                                        // VALIDAR SI ELIMINARON LA FECHA DE BAJA AL EDITAR ENTONCES Y ES NULL PONEMOS VACIO
                                        if (this.formTableAlfanumerico.fechaBaja === null) {
                                            this.formTableAlfanumerico.fechaBaja = '' 
                                        }

                                        this.dataTableModalRangosCuota.push({'numAlfanumerico': this.formTableAlfanumerico.numAlfanumerico, 'fechaAlta': this.formTableAlfanumerico.fechaAlta, 'fechaBaja': this.formTableAlfanumerico.fechaBaja})
                                        this.formTableAlfanumerico.numAlfanumerico = ''
                                        this.formTableAlfanumerico.fechaAlta = ''
                                        this.formTableAlfanumerico.fechaBaja = ''
                                        var listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                                            return a.numAlfanumerico.toString().toLowerCase() > b.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                                        });
                                        this.dataTableModalRangosCuota = listaOrdenada
                                        this.getListaRangosCuotasFiltrado()
                                        this.formIsEdit = false
                                    }

                            }).catch((err) => {
                                console.log(err)
                                this.$notify.info({title: err.message, customClass: 'notificacionWarning'})
                            }) 
                        }
                    }).catch((err) => {
                        console.log(err)
                        this.$notify.info({title: err.message, customClass: 'notificacionWarning'})
                    })                    
                } else {
                    this.$notify.error({title: this.$t('message.erroresValidacion'), customClass: 'notificacionError'})
                }
            })
        },
        eliminarRegistroCuotasRangos (row) {
            this.$confirm(this.$t('contratos.dialogTable.deleteModaltable'), this.$t('message.title.confirmacion'), {
            cancelButtonText: this.$t('button.cancelar'),
            confirmButtonText: this.$t('button.aceptar'),
            type: 'warning'
            }).then(() => {
                var desdeF = row.numAlfanumerico
                var hastaF = row.fechaAlta
                var cuotaF = row.fechaBaja
                var listaFiltrada = this.dataTableModalRangosCuota.filter( function(item) {
                    return item.numAlfanumerico != desdeF || item.fechaAlta != hastaF || item.fechaBaja != cuotaF
                })
                this.dataTableModalRangosCuota = listaFiltrada
                this.getListaRangosCuotasFiltrado()
            }).catch(() => {})
        },
        newRangosCuotas () {
            this.formIsEdit = false
            this.modalRangosCuota.showForm = true
            this.formTableAlfanumerico.numAlfanumerico = ''
            this.formTableAlfanumerico.fechaAlta = ''
            this.formTableAlfanumerico.fechaBaja = ''
        },
        editarRegistroCuotasRangos(row) {
            this.formIsEdit = true
            this.modalRangosCuota.showForm = true
            this.formTableAlfanumerico.numAlfanumericoSelected = row.numAlfanumerico.toString()
            this.formTableAlfanumerico.fechaAltaSelected = row.fechaAlta.toString()
            this.formTableAlfanumerico.fechaBajaSelected = row.fechaBaja.toString()

            this.formTableAlfanumerico.numAlfanumerico = row.numAlfanumerico.toString()
            this.formTableAlfanumerico.fechaAlta = row.fechaAlta.toString()
            this.formTableAlfanumerico.fechaBaja = row.fechaBaja.toString()
        },
        sortChanged ({ column, prop, order }) {
            this.modalRangosCuota.currentPage = 1
            this.modalRangosCuota.currentSortProp = prop
            this.modalRangosCuota.currentSortOrder = order
            if (this.loadCaudalCuotasFirstTime) {
                // this.getRemitenteDataAlfanumericos()
                this.loadCaudalCuotasFirstTime = false
            }
            this.ordenarTabla(this.modalRangosCuota.currentSortProp, this.modalRangosCuota.currentSortOrder)
            this.getListaRangosCuotasFiltrado()
        },
        ordenarTabla (prop, order) {
            var listaOrdenada
            if (order == 'ascending') {
                if (prop === 'numAlfanumerico') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.numAlfanumerico.toString().toLowerCase() > b.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                    })
                    
                } else if (prop === 'fechaAlta') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.fechaAlta.toString().toLowerCase() > b.fechaAlta.toString().toLowerCase() ? 1 : -1
                    })
                }else if (prop === 'fechaBaja') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.fechaBaja.toString().toLowerCase() > b.fechaBaja.toString().toLowerCase() ? 1 : -1
                    })
                } else {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.numAlfanumerico.toString().toLowerCase() > b.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                    })
                }
            }else if (order == 'descending') {
                if (prop === 'numAlfanumerico') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return b.numAlfanumerico.toString().toLowerCase() > a.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                    })
                } else if (prop === 'fechaAlta') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return b.fechaAlta.toString().toLowerCase() > a.fechaAlta.toString().toLowerCase() ? 1 : -1
                    })
                }else if (prop === 'fechaBaja') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return b.fechaBaja.toString().toLowerCase() > a.fechaBaja.toString().toLowerCase() ? 1 : -1
                    })
                } else {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return b.numAlfanumerico.toString().toLowerCase() > a.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                    })
                }
            } else {
                if (prop === 'numAlfanumerico') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.numAlfanumerico.toString().toLowerCase() > b.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                    }) 
                } else if (prop === 'fechaAlta') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.fechaAlta.toString().toLowerCase() > b.fechaAlta.toString().toLowerCase() ? 1 : -1
                    })
                }else if (prop === 'fechaBaja') {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.fechaBaja.toString().toLowerCase() > b.fechaBaja.toString().toLowerCase() ? 1 : -1
                    })
                } else {
                    listaOrdenada = this.dataTableModalRangosCuota.slice().sort(function(a, b) {
                        return a.numAlfanumerico.toString().toLowerCase() > b.numAlfanumerico.toString().toLowerCase() ? 1 : -1
                    })
                }
            }
            this.dataTableModalRangosCuota = listaOrdenada
        },
        dblClickEditarCuotasRangos () {

        },
        handleCurrentChangeCuotasRangos (val) {
            // this.currentRow = val
        },
        getRemitenteDataAlfanumericos () {
            // this.remitentesModel.COD_NUMALFANUM
            getRemitenteDataAlfanumericos(this.codNumalfanum).then((resp) => {

                resp.data.forEach((value, index) => {
                    var d1 = value.FEC_ALTA ? formatDateToFormat(value.FEC_ALTA, "DD/MM/YYYY") : ''
                    var d2 = value.FEC_BAJA ? formatDateToFormat(value.FEC_BAJA, "DD/MM/YYYY") : ''
                    this.dataTableModalRangosCuota.push({'numAlfanumerico': value.NUM_ALFANUMERICO, 
                        'fechaAlta': d1, 
                        'fechaBaja': d2});
                });
                this.InitialdataTableModalRangosCuota = this.dataTableModalRangosCuota
                this.dataTableModalRangosCuotaOriginal = this.dataTableModalRangosCuota
                // this.$emit('handleSaveAllTable', this.dataTableModalRangosCuota)

                //ENVIAMOS EL TAMAÑO DE LA LISTA PARA VALIDAR LOS ALFANUMERICOS
                if (this.dataTableModalRangosCuota.length > 0) {
                    this.$emit('tableModalSize', this.dataTableModalRangosCuota.length)
                }

                this.getListaRangosCuotasFiltrado()
            }).catch((err) => {
                console.log(err)
                this.$notify.info({title: err.message, customClass: 'notificacionWarning'})
            })
        },
        handleSizeChangeModalRangosCuotas (val) {
            //alert('SIZE: '+this.modalRangosCuota.currentPage)
            this.modalRangosCuota.numRegisters = val
            this.getListaRangosCuotasFiltrado()
        },
        handleChangesPageModalRangosCuotas () {
            //alert('PAGE: '+this.modalRangosCuota.currentPage)
            this.getListaRangosCuotasFiltrado()
        },
        getListaRangosCuotasFiltrado () {
            this.modalRangosCuota.totalItems = this.dataTableModalRangosCuota.length
            var hPage = ( this.modalRangosCuota.numRegisters * this.modalRangosCuota.currentPage )
            var dPage =  hPage - this.modalRangosCuota.numRegisters
            this.dataTableModalRangosCuotaForPage = []
            for (var i = dPage; i < hPage; i++) {
                try{
                    var numAlfanumerico = this.dataTableModalRangosCuota[i].numAlfanumerico;
                    var fechaAlta = this.dataTableModalRangosCuota[i].fechaAlta;
                    var fechaBaja = this.dataTableModalRangosCuota[i].fechaBaja;
                    this.dataTableModalRangosCuotaForPage.push({'numAlfanumerico': numAlfanumerico, 'fechaAlta': fechaAlta , 'fechaBaja': fechaBaja});
                }catch (e){
                    console.log(e)
                }   
            }
            
            // SI LA PAGINA ES MAYOR A 1 Y SE ELIMINA EL ULTIMO REGISTRO DE UNA PAGINA DEBEMOS REDUCIR EL HPAGE
            if (this.dataTableModalRangosCuotaForPage.length == 0 && this.modalRangosCuota.currentPage !== 1) {
                this.modalRangosCuota.currentPage--
                this.getListaRangosCuotasFiltrado()
            }
        }
        //  FIN RANGOS CUOTAS
    },
    created: function () {
    },
    mounted: function () {
    },
    watch: { 
        visibility: function(newVal, oldVal) {
          this.modalRangosCuota.dialogVisible = newVal
        },
        codNumalfanum: function (newVal, oldVal) {
            if (newVal !== '' || newVal !== null) {
                this.getRemitenteDataAlfanumericos()
            }
        }
    },
}
</script>
