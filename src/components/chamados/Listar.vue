<template>
    <q-pull-to-refresh pull-message="Puxe para baixo para atualizar" release-message="solte vagabundo" refresh-message="atualizando..."  refresh-icon :handler="refresher" >
        <div >
            </q-btn>
            <div class="row shadow-10 pb">
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-input v-model="buscaId" type="number" float-label="ID" @change="buscar" />
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-input v-model="buscaAssunto" type="text" float-label="Assunto" @change="buscar" />
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-select toggle v-model="buscaResponsavel" filter float-label="Responsável" :options="listaResponsaveis" @change="buscar"/>
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-select toggle v-model="buscaTipoChamado" filter float-label="Tipo do Chamado" :options="listaTiposChamado" @change="buscar"/>
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-select toggle v-model="buscaStatus" filter float-label="Status" :options="listaStatus" @change="buscar"/>
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-datetime  v-model="buscaDataRecebimento" type="date" format="DD/MM/YYYY" float-label="Data Recebimento" @change="buscar" />
                </div>
            </div>
            <div class="row mt2 shadow-10">
                <q-data-table :data="chamadosFiltrados" :config="config" :columns="columns">
                    <template slot="col-Id" scope="cell">
                        <router-link class="text-indigo" :to="'/editar/'+cell.data">{{ cell.data }}</router-link>
                    </template>
                     <template slot="col-StatusChamado" scope="cell">
                        <q-select class="mt0" v-model="cell.data.Id" filter :options="listaStatusTable" @change="trocaStatus(cell.data.Id, cell.row.Id)"/>
                    </template>                    
                    <template slot="col-ResponsavelChamado" scope="cell">
                        <q-select class="mt0" v-model="cell.data.Id" filter :options="listaResponsaveis" @change="trocaResponsavel(cell.data.Id, cell.row.Id)"/>
                    </template>
                    <template slot="col-TipoChamado" scope="cell">
                        <q-select class="mt0" v-model="cell.data.Id" filter :options="listaTiposChamado" @change="trocaTipoChamado(cell.data.Id, cell.row.Id)"/>
                    </template>                    
                    <template slot="selection" scope="props">
                        <q-btn flat color="primary" @click="excluir(props)">
                            <q-icon name="delete" />
                        </q-btn>
                    </template>
                </q-data-table>             
            </div>
        </div>
    </q-pull-to-refresh>
</template>
<script>
    import {
        Toast,
        QDataTable,
        QPullToRefresh,
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
                    rowHeight: '3.6rem',
                    selection: 'multiple',
                    messages: {
                        noData: '<q-icon name="warning"/> Sem dados para mostrar.'
                    },
                    labels: {
                        selected: {
                            singular: 'item selecionado.',
                            plural: 'itens selecionados.'
                        }
                    }
                },
                columns: [
                    { label: 'ID', field: 'Id', width: '30px', sort: true, type: 'number' },
                    { label: 'Assunto', field: 'Assunto', width: '140px' },
                    { label: 'Remetente', field: 'NomeCliente', width: '100px', sort: true, type: 'string' },
                    { label: 'Status', field: 'StatusChamado', width: '60px' },
                    { label: 'Responsável', field: 'ResponsavelChamado', width: '50px' },
                    { label: 'Tipo de Chamado', field: 'TipoChamado', width: '60px' },
                    { label: 'Data de Recebimento', field: 'DataRecebimento', width: '60px', type: 'date', sort: true, format (value, row) { return new Date(value).toLocaleString() } },
                    { label: 'Prazo Final', field: 'PrazoFinal', width: '60px', type: 'date', sort: true, format (value, row) { return new Date(value).toLocaleString() } }
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
                listaStatusTable: [],
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
            QPullToRefresh,
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
                        chamado.TipoChamado = { Nome: '', Id: null }
                    }
                    if (!chamado.ResponsavelChamado) {
                        chamado.ResponsavelChamado = { Nome: '', Id: null }
                    }
                })
                this.chamadosFiltrados = this.chamados.slice(0) // copia do vetor
            })

            axios.get(url + 'StatusChamados')
            .then(response => {
                this.listaStatus = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaStatusTable = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                // this.listaStatusTable = this.listaStatus
                this.listaStatus.unshift({ label: '', value: null })
            })

            axios.get(url + 'ResponsavelChamados')
            .then(response => {
                this.listaResponsaveis = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaResponsaveis.unshift({ label: '', value: null })
            })

            axios.get(url + 'TipoChamados')
            .then(response => {
                this.listaTiposChamado = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaTiposChamado.unshift({ label: '', value: null })
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
                            Toast.create.negative({ html: 'Problemas na Exclusão do Chamado ' + row.data.Id })
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
                                    (!self.buscaAssunto || chamado.Assunto.toUpperCase().indexOf(self.buscaAssunto.toUpperCase()) >= 0) &&
                                    (!self.buscaResponsavel || chamado.ResponsavelChamado.Id == self.buscaResponsavel) &&
                                    (!self.buscaTipoChamado || chamado.TipoChamado.Id == self.buscaTipoChamado) &&
                                    (!self.buscaStatus || chamado.StatusChamado.Id == self.buscaStatus) &&
                                    (!self.buscaDataRecebimento || chamado.DataRecebimento.substring(0,10) == self.buscaDataRecebimento.substring(0,10))                                    
                                    /* eslint-enable */
                               )
                    }
                )
            },
            refresher (done) {
                setTimeout(() => {
                    done()
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
                                chamado.TipoChamado = { Nome: '', Id: null }
                            }
                            if (!chamado.ResponsavelChamado) {
                                chamado.ResponsavelChamado = { Nome: '', Id: null }
                            }
                        })
                        this.chamadosFiltrados = this.chamados.slice(0) // copia do vetor
                    })
                }, 1000)
            },
            trocaResponsavel (value, id) {
                let self = this
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaResponsavelChamado?id=' + id + '&responsavelid=' + value)
                .then(function (response) {
                    if (response.status === 200) {
                        self.chamadosFiltrados.forEach(function (chamado) {
                            if (chamado.Id === id) {
                                chamado.ResponsavelChamado.Id = value
                            }
                        })
                        Toast.create.positive({ html: 'Responsavel Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Responsavel' })
                    }
                })
            },
            trocaTipoChamado (value, id) {
                let self = this
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaTipoChamado?id=' + id + '&tipoChamadoId=' + value)
                .then(function (response) {
                    if (response.status === 200) {
                        self.chamadosFiltrados.forEach(function (chamado) {
                            if (chamado.Id === id) {
                                chamado.TipoChamado.Id = value
                            }
                        })
                        Toast.create.positive({ html: 'Tipo de Chamado Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Tipo de Chamado' })
                    }
                })
            },
            trocaStatus (value, id) {
                let self = this
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaStatusChamado?id=' + id + '&statusid=' + value)
                .then(function (response) {
                    if (response.status === 200) {
                        self.chamadosFiltrados.forEach(function (chamado) {
                            if (chamado.Id === id) {
                                chamado.StatusChamado.Id = value
                            }
                        })
                        Toast.create.positive({ html: 'Status do Chamado Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Status do Chamado' })
                    }
                })
            }
        }
    }
</script>

<style lang="stylus">
    .p3
        padding 0 .75rem
    .mt2
        margin-top 2rem
    .mt0
        margin-top 0rem           
    .pb
        padding-bottom 1.8rem
</style>
