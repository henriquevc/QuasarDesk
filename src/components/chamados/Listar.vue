<template>
    <div >
        <div class="row">    
            <div class="col-lg-2 col-md-4 p3">
                <q-input v-model="buscaId" type="number" float-label="ID" @change="buscar" />
            </div>
            <div class="col-lg-2 col-md-4 p3">
                <q-input v-model="buscaAssunto" type="text" float-label="Assunto" @change="buscar" />
            </div>
            <div class="col-lg-2 col-md-4 p3">
                <q-select toggle v-model="buscaResponsavel" filter float-label="Responsável" :options="listaResponsaveis" @change="buscar"/>
            </div>
            <div class="col-lg-2 col-md-4 p3">
                <q-select toggle v-model="buscaTipoChamado" filter float-label="Tipo do Chamado" :options="listaTiposChamado" @change="buscar"/>
            </div>
            <div class="col-lg-2 col-md-4 p3">
                <q-select toggle v-model="buscaStatus" filter float-label="Status" :options="listaStatus" @change="buscar"/>
            </div>
            <div class="col-lg-2 col-md-4 p3">
                <q-datetime  v-model="buscaDataRecebimento" type="date" format="DD/MM/YYYY" float-label="Data Recebimento" @change="buscar" />
            </div>
        </div>
        <div class="row">       
            <q-data-table :data="chamadosFiltrados" :config="config" :columns="columns">
                <template slot="col-Id" scope="cell">
                    <a :href="'/editar/'+cell.data">{{cell.data}}</a>
                </template>                
                <template slot="col-StatusChamado" scope="cell">
                    <span>{{cell.data.Nome}}</span>
                </template>
                <template slot="col-ResponsavelChamado" scope="cell">
                    <span>{{cell.data.Nome}}</span>
                </template>
                <template slot="col-TipoChamado" scope="cell">
                    <span>{{cell.data.Nome}}</span>
                </template>
                <template slot="selection" scope="props">
                    <q-btn flat color="primary" @click="excluir(props)">
                        <q-icon name="delete" />
                    </q-btn>
                </template>
            </q-data-table>             
        </div>
    </div>
</template>
<script>
    import {
    Toast,
    QDataTable,
    QField,
    QInput,
    QCheckbox,
    QSelect,
    QSlider,
    QBtn,
    QIcon,
    QTooltip,
    QCollapsible,
    QDatetime
    } from 'quasar'
    import axios from 'Axios'
    import 'quasar-extras/material-icons'

    export default {
        data () {
            return {
                config: {
                    rowHeight: '4rem',
                    selection: 'multiple'
                },
                columns: [
                    { label: 'ID', field: 'Id', width: '50px', sort: true, type: 'number' },
                    { label: 'Assunto', field: 'Assunto', width: '140px' },
                    { label: 'Remetente', field: 'NomeCliente', width: '100px', sort: true, type: 'string' },
                    { label: 'Status', field: 'StatusChamado', width: '60px' },
                    { label: 'Responsável', field: 'ResponsavelChamado', width: '100px' },
                    { label: 'Tipo de Chamado', field: 'TipoChamado', width: '100px' },
                    { label: 'Data de Recebimento', field: 'DataRecebimento', width: '100px', type: 'date', sort: true, format (value, row) { return new Date(value).toLocaleString() } }
                ],
                chamados: [],
                chamadosFiltrados: [],
                urlEditar: '',
                buscaId: null,
                buscaAssunto: '',
                buscaResponsavel: '',
                buscaDataRecebimento: '',
                buscaTipoChamado: '',
                buscaStatus: '',
                listaStatus: [],
                listaResponsaveis: [],
                listaTiposChamado: []
            }
        },
        components: {
            QInput,
            QSelect,
            QDatetime,
            QDataTable,
            QField,
            QCheckbox,
            QSlider,
            QBtn,
            QIcon,
            QTooltip,
            QCollapsible
        },
        created () {
            let url = 'http://servagilus.dyndns.org:9801/api/'
            let tipoStatus = this.$route.params.tipoStatus
            if (!tipoStatus) {
                tipoStatus = 'trabalhaveis'
            }
            axios.get(url + 'Chamados?tipoStatus=' + tipoStatus)
                .then(response => {
                    this.chamados = response.data
                    this.chamados.forEach(function (chamado) {
                        if (!chamado.TipoChamado) {
                            chamado.TipoChamado = { Nome: '' }
                        }
                        if (!chamado.ResponsavelChamado) {
                            chamado.ResponsavelChamado = { Nome: '' }
                        }
                    })
                    this.chamadosFiltrados = this.chamados.slice(0) // copia do vetor
                })

            axios.get(url + 'StatusChamados')
                .then(response => {
                    this.listaStatus = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaStatus.unshift({ label: '', value: '' })
                })

            axios.get(url + 'ResponsavelChamados')
                .then(response => {
                    this.listaResponsaveis = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaResponsaveis.unshift({ label: '', value: '' })
                })

            axios.get(url + 'TipoChamados')
                .then(response => {
                    this.listaTiposChamado = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaTiposChamado.unshift({ label: '', value: '' })
                })
        },
        methods: {
            excluir (chamado) {
                let url = 'http://servagilus.dyndns.org:9801/api/'
                chamado.rows.forEach(row => {
                    let self = this
                    axios.delete(url + 'Chamados/' + row.data.Id).then(response => {
                        if (response.status === 200) {
                            self.chamadosFiltrados.splice(row.index, 1)
                            Toast.create.positive({ html: 'Chamado ' + row.data.Id + ' Excluído' })
                        }
                        else {
                            Toast.create.negative({ html: 'Chamado ' + row.data.Id + ' Excluído' })
                        }
                    })
                })
            },
            buscar () {
                let self = this
                self.chamadosFiltrados = self.chamados.filter(
                    function (chamado) {
                        return (
                                    /* eslint-disable */
                                    (!self.buscaId || chamado.Id == self.buscaId) &&
                                    (!self.buscaAssunto || chamado.Assunto.indexOf(self.buscaAssunto) >= 0) &&
                                    (!self.buscaResponsavel || chamado.ResponsavelChamado.Id == self.buscaResponsavel) &&
                                    (!self.buscaTipoChamado || chamado.TipoChamado.Id == self.buscaTipoChamado) &&
                                    (!self.buscaStatus || chamado.StatusChamado.Id == self.buscaStatus) &&
                                    (!self.buscaDataRecebimento || chamado.DataRecebimento.substring(0,10) == self.buscaDataRecebimento.substring(0,10))                                    
                                    /* eslint-enable */
                               )
                    }
                )
            }
        }
    }
</script>

<style>
    .p3{
        padding: 0 1rem;
    }
        
</style>
