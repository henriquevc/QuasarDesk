<template>
    <div>
        <q-tabs align="justify" v-model="selectedTab">
        <q-tab slot="title" :name="status.label" :icon="status.icone" v-for="status in listaStatus" key @select="trocaStatus(status.value)">{{status.label}}</q-tab>
        </q-tabs>
        <div class="row">
            <div class="col-lg-4 col-md-12">
                <q-card>
                    <q-input stack-label="Assunto" :disable="true" v-model="chamado.Assunto"/>
                    <q-input stack-label="Email" :disable="true" v-model="chamado.Email"/>
                    <q-select v-model="responsavelId" filter float-label="Responsávl" :options="listaResponsaveis" @change="trocaResponsavel"/>
                    <q-select v-model="tipoChamadoId" filter float-label="Tipo Chamado" :options="listaTiposChamado" @change="trocaTipoChamado"/>
                    <q-input stack-label="Prazo Final" :disable="true" v-model="chamado.PrazoFinal"/>
                    <q-input stack-label="Prazo Finalização" :disable="true" v-model="chamado.DataFinalizacao"/>
                    <q-input stack-label="Nome Cliente" :disable="true" v-model="chamado.NomeCliente"/>
                </q-card>  
            </div>
            <div class="col-lg-8 col-md-12">
                <q-card v-for="mensagem in chamado.MensagensChamado" key>
                    {{mensagem.CorpoEmail}}                     
                </q-card>  
            </div>

        </div>
    </div>
</template>

<script>
    import axios from 'Axios'
    import {
        Toast,
        QTabs,
        QBtn,
        QTab,
        QTabPane,
        QCard,
        QInput,
        QSelect
        } from 'quasar'

    export default {
        components: {
            QTabs,
            QBtn,
            QTab,
            QTabPane,
            QCard,
            QInput,
            QSelect
        },
        data () {
            return {
                chamado: {},
                listaStatus: [],
                listaResponsaveis: [],
                listaTiposChamado: [],
                selectedTab: null,
                tipoChamadoId: null,
                responsavelId: null,
                responsavel: {},
                tipoChamado: {},
                statusChamado: {}
            }
        },
        created () {
             axios.get('http://servagilus.dyndns.org:9801/api/Chamados/' + this.$route.params.id)
             .then(response => {
                this.chamado = response.data

                axios.get('http://servagilus.dyndns.org:9801/api/StatusChamados')
                .then(response => {
                    this.listaStatus = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaStatus[0].icone = 'message'
                    this.listaStatus[1].icone = 'build'
                    this.listaStatus[2].icone = 'done'
                    this.listaStatus[3].icone = 'stop'
                    this.selectedTab = this.chamado.StatusChamado.Nome
                    this.tipoChamadoId = this.chamado.TipoChamado.Id
                    this.responsavelId = this.chamado.ResponsavelChamado.Id
                })
                axios.get('http://servagilus.dyndns.org:9801/api/ResponsavelChamados')
                .then(response => {
                    this.listaResponsaveis = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                })
                axios.get('http://servagilus.dyndns.org:9801/api/TipoChamados')
                .then(response => {
                    this.listaTiposChamado = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                })
            })
        },
        methods: {
            trocaResponsavel (value) {
                let self = this
                self.responsavel = self.listaResponsaveis.find(function (responsaveis) {
                    return (responsaveis.value === value)
                })
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaResponsavelChamado?id=' + this.chamado.Id + '&responsavelid=' + value)
                .then(function (response) {
                    if (response.status === 200) {
                        Toast.create.positive({ html: 'Responsavel Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Responsavel' })
                    }
                })
            },
            trocaTipoChamado (value) {
                let self = this
                self.tipoChamado = self.listaTiposChamado.find(function (responsaveis) {
                    return (responsaveis.value === value)
                })
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaTipoChamado?id=' + this.chamado.Id + '&tipoChamadoId=' + value)
                .then(function (response) {
                    if (response.status === 200) {
                        Toast.create.positive({ html: 'Tipo de Chamado Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Tipo de Chamado' })
                    }
                })
            },
            trocaStatus (value) {
                let self = this
                if (value !== self.chamado.StatusId) {
                    self.statusChamado = self.listaStatus.find(function (responsaveis) {
                        return (responsaveis.value === value)
                    })
                    axios.put('http://servagilus.dyndns.org:9801/api/TrocaStatusChamado?id=' + this.chamado.Id + '&statusid=' + value)
                    .then(function (response) {
                        if (response.status === 200) {
                            Toast.create.positive({ html: 'Status do Chamado Atualizado' })
                        }
                        else {
                            Toast.create.negative({ html: 'Problemas na Atualização do Status do Chamado' })
                        }
                    })
                    self.chamado.StatusId = value
                }
            }
        }
    }
</script>
