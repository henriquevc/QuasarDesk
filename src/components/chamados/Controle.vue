<template>
    <q-pull-to-refresh pull-message="Puxe para baixo para atualizar" release-message="Solte para atualizar" refresh-message="Atualizando..."  refresh-icon :handler="refresher" >
        <div>
            <div class="row shadow-10 pb bg-white">
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-input v-model="buscaId" type="number" float-label="ID" @change="buscar" />
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-select ref="responsavel" @focus="selectOptionFocus('responsavel')" autofocusfilter toggle v-model="buscaResponsavel" filter float-label="Responsável" :options="listaResponsaveis" @change="buscar"/>
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-select ref="orgao" @focus="selectOptionFocus('orgao')" autofocusfilter toggle v-model="buscaOrgaoChamado" filter float-label="Órgão do Chamado" :options="listaOrgaoChamado" @change="buscar"/>
                </div>                
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-select ref="status" @focus="selectOptionFocus('status')" autofocusfilter toggle v-model="buscaStatus" filter float-label="Status" :options="listaStatus" @change="buscar"/>
                </div>
                <div class="col-xl-2 col-lg-3 col-md-4 col-xs-12 p3">
                    <q-datetime v-model="buscaDataRecebimento" type="date" format="DD/MM/YYYY" float-label="Data Recebimento" @change="buscar" />
                </div>
            </div>        
            <div class="row mt2 shadow-10 bg-white">
                <q-pagination v-model="page" :max="maxPages" @change="mudaMes" class="m"/>
                    <table class="q-table striped-odd responsive compact" style="width: 100%; font-size:12px">
                        <thead>
                            <tr>
                                <th :class="coluna.flex" v-for="coluna in columns" :key="coluna">{{coluna.label}}</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="chamado in chamadosFiltrados" :key="chamado">
                                <td><router-link class="text-teal bold" :id="chamado.Id" :to="'/editar/'+chamado.Id">{{ chamado.Id }}</router-link></td>
                                <td><div>{{chamado.Assunto}}</div></td>
                                <td><div>{{chamado.Cliente.Nome}}</div></td>
                                <td><div>{{chamado.StatusChamado.Nome}}</div></td>
                                <td><div>{{chamado.ResponsavelChamado.Nome}}</div></td>
                                <td><div>{{chamado.OrgaoChamado.Nome}}</div></td>
                                <td><div>{{chamado.DataRecebimento.toLocaleDateString()}}</div></td>
                                <td><div>{{chamado.PrazoFinal.toLocaleDateString()}}</div></td>
                            </tr>
                        </tbody>
                    </table>   
                <q-pagination v-model="page" :max="maxPages" @change="mudaMes"/>
            </div>
        </div>
    </q-pull-to-refresh>
</template>
<script>
    import { QPagination, QDataTable, QPullToRefresh, QField, QInput, QCheckbox, QSelect, QSlider, QBtn, QIcon, QTooltip, QCollapsible, QDatetime } from 'quasar'
    import axios from 'Axios'
    import 'quasar-extras/material-icons'

    export default {
        data () {
            return {
                columns: [
                    { label: 'ID' },
                    { label: 'Assunto' },
                    { label: 'Cliente' },
                    { label: 'Status' },
                    { label: 'Responsável' },
                    { label: 'Órgão' },
                    { label: 'Data de Recebimento' },
                    { label: 'Prazo Final' }
                ],
                chamados: [],
                chamadosFiltrados: [],
                page: 1,
                maxPages: 12,
                buscaId: null,
                buscaResponsavel: '',
                buscaDataRecebimento: '',
                buscaOrgaoChamado: '',
                buscaStatus: '',
                listaStatus: [],
                listaResponsaveis: [],
                listaOrgaoChamado: []
            }
        },
        components: { QPagination, QInput, QSelect, QDatetime, QDataTable, QField, QCheckbox, QPullToRefresh, QSlider, QBtn, QIcon, QTooltip, QCollapsible },
        created () {
            let self = this
            let url = 'http://servagilus.dyndns.org:9801/api/'
            let tipoStatus = this.$route.params.tipoStatus
            if (!tipoStatus) {
                tipoStatus = 'bases'
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
                    if (!chamado.OrgaoChamado) {
                        chamado.OrgaoChamado = { Nome: '', Id: null }
                    }
                    if (!chamado.Cliente) {
                        chamado.Cliente = { Nome: '', Id: null }
                    }
                })
                this.chamadosFiltrados = this.chamados.filter(chamado => {
                    let datainicial = new Date(Date.now())
                    let datafinal = new Date(Date.now())
                    datainicial.setDate(1)
                    datafinal.setDate(1)
                    datafinal.setMonth(datafinal.getMonth() + 1)
                    datafinal.setDate(-1)
                    self.page = datainicial.getMonth() + 1
                    chamado.DataRecebimento = new Date(chamado.DataRecebimento)
                    chamado.PrazoFinal = new Date(chamado.PrazoFinal)
                    return (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)
                })
            })
            axios.get(url + 'StatusChamados')
            .then(response => {
                this.listaStatus = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaStatusTable = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
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

            axios.get(url + 'OrgaoChamados')
            .then(response => {
                this.listaOrgaoChamado = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaOrgaoChamado.unshift({ label: '', value: null })
            })
        },
        methods: {
            selectOptionFocus (refs) {
                if (typeof this.$refs[refs].$children === 'undefined') {
                    this.$refs[refs][0].$children[0].$children[0].$children[0].focus()
                }
                else {
                    this.$refs[refs].$children[0].$children[0].$children[0].focus()
                }
            },
            refresher (done) {
                setTimeout(() => {
                    done()
                    let url = 'http://servagilus.dyndns.org:9801/api/'
                    let tipoStatus = this.$route.params.tipoStatus
                    if (!tipoStatus) {
                        tipoStatus = 'bases'
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
                            if (!chamado.OrgaoChamado) {
                                chamado.OrgaoChamado = { Nome: '', Id: null }
                            }
                        })
                        this.chamadosFiltrados = this.chamados.filter(function (chamado) {
                            let datainicial = new Date(Date.now())
                            let datafinal = new Date(Date.now())
                            datainicial.setDate(1)
                            datafinal.setDate(1)
                            datafinal.setMonth(datafinal.getMonth() + 1)
                            datafinal.setDate(-1)
                            self.page = datainicial.getMonth() + 1
                            return (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)
                        })
                    })
                }, 1000)
            },
            mudaMes (mes) {
                this.chamadosFiltrados = this.chamados.filter(function (chamado) {
                    let datainicial = new Date(Date.now())
                    let datafinal = new Date(Date.now())
                    datainicial.setDate(1)
                    datainicial.setMonth(mes - 1)
                    datafinal.setDate(1)
                    datafinal.setMonth(mes)
                    datafinal.setDate(-1)
                    return (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)
                })
            },
            buscar () {
                let self = this
                self.chamadosFiltrados = self.chamados.filter(function (chamado) {
                    let datainicial = new Date(Date.now())
                    let datafinal = new Date(Date.now())
                    datainicial.setDate(1)
                    datainicial.setMonth(self.page - 1)
                    datafinal.setDate(1)
                    datafinal.setMonth(self.page)
                    datafinal.setDate(-1)
                    return (
                        /* eslint-disable */
                        (!self.buscaId || chamado.Id == self.buscaId && (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)) &&
                        (!self.buscaResponsavel || chamado.ResponsavelChamado.Id == self.buscaResponsavel && (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)) &&
                        (!self.buscaOrgaoChamado || chamado.OrgaoChamado.Id == self.buscaOrgaoChamado && (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)) &&
                        (!self.buscaStatus || chamado.StatusChamado.Id == self.buscaStatus && (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)) &&
                        (!self.buscaDataRecebimento || chamado.DataRecebimento.substring(0,10) == self.buscaDataRecebimento.substring(0,10) && (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)) &&
                        (new Date(chamado.DataRecebimento) >= datainicial) && (new Date(chamado.DataRecebimento) <= datafinal)
                        /* eslint-enable */
                    )
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
    .pb
        padding-bottom 1.8rem
    .bold
        font-weight bold
    tr
        backgroud-color white
</style>
