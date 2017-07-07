<template>
    <div>
        <q-tabs align="justify" class="mb-1 shadow-10" style="margin-bottom: 16px;" v-if="chamado.StatusChamado" v-model="chamado.StatusChamado.Nome">
            <q-tab slot="title" :name="status.label" :icon="status.icone" v-for="status in listaStatus" key @select="trocaStatus(status.value, status.label)">{{status.label}}</q-tab>
        </q-tabs>
        <div class="row">
            <div class="col-lg-4 col-md-12">
                <q-card class="pa-1 shadow-10 bg-white">
                    <q-input stack-label="Assunto" readonly v-model="chamado.Assunto"/>
                    <q-input stack-label="E-mail" readonly v-model="chamado.Email"/>
                    <q-select v-model="chamado.ResponsavelId" filter float-label="Responsável" :options="listaResponsaveis" @change="trocaResponsavel"/>
                    <q-select v-model="chamado.TipoChamadoId" filter float-label="Tipo Chamado" :options="listaTiposChamado" @change="trocaTipoChamado"/>
                    <q-input stack-label="Prazo Final" readonly v-model="chamado.PrazoFinal"/>
                    <q-input stack-label="Prazo Finalização" :disable="true" v-model="chamado.DataFinalizacao"/>
                    <q-input stack-label="Nome Cliente" readonly v-model="chamado.NomeCliente"/>
                </q-card>  
            </div>
            <q-fab direction="up" class="fixed" style="right: 2rem; bottom: .5rem; z-index: 1" color="primary" icon="keyboard_arrow_up">
                  <q-fab-action round color="red" v-show="!chamado.Excluido" @click="excluir" icon="delete"/>   
                  <q-fab-action round color="tertiary" v-show="chamado.Excluido" @click="excluir" icon="undo"/> 
                  <q-fab-action round color="info" v-show="botaoAtivo" @click="btnresponder" icon="reply"/>                    
            </q-fab>                                
            <div class="col-lg-8 col-md-12">
                <q-card v-for="mensagem in chamado.MensagensChamado" key class="bg-white shadow-10" style="margin-bottom: 1rem;">
                    <q-card-title class="bg-secondary text-white">
                        #{{mensagem.Id}} - {{ mensagem.DataRecebimento }}
                        <q-icon color="white" slot="right" name="attach file" v-if="mensagem.AnexosMensagem.length > 0">
                                <q-popover ref="popover">
                                    <q-list link class="no-border">
                                        <q-item link tag="a" exact :href="url+'AnexosMensagens/'+anexo.Id" v-for="anexo in mensagem.AnexosMensagem" key>
                                            <q-item-main :label="anexo.NomeAnexo" />
                                        </q-item>
                                    </q-list>
                                </q-popover>
                        </q-icon>                        
                    </q-card-title>
                    <q-card-main>
                       <div class="pa-1" v-html="mensagem.CorpoEmail"/>
                    </q-card-main>                
                </q-card>
                <q-card>
                    <div class="elevation-10">
                        <div class="mb-1 bg-white" id="comentario" v-show="!botaoAtivo" style=" height: 324px;">
                            <vue-editor v-model="content" v-show="!botaoAtivo" id="editorHtml" style=" height: 285px;"> </vue-editor>
                        </div>
                    </div><!--col-12-->
                </q-card>          
                <div class="mb-1 mt-1">
                    <q-btn flat class="bg-positive text-white" v-show="!botaoAtivo" @click="enviarMensagem()">Enviar <q-icon right name="send"/></q-btn>
                    <q-btn flat class="bg-negative text-white" v-show="!botaoAtivo" @click="destruirResposta()">Cancelar <q-icon right name="block"/></q-btn>
                </div><!--col-12-->                         
            </div>            
        </div>
    </div>
</template>

<script>
    import axios from 'Axios'
    import { VueEditor } from 'vue2-editor'
    import 'quasar-extras/material-icons'
    import {
        Toast,
        QFab,
        QFabAction,
        QTabs,
        QItem,
        QItemMain,
        QPopover,
        QIcon,
        QList,
        QBtn,
        QTab,
        QTabPane,
        QCard,
        QCardMain,
        QCardTitle,
        QInput,
        QSelect,
        QCollapsible,
        QSideLink
    } from 'quasar'

    export default {
        components: {
            QFab,
            QFabAction,
            QTabs,
            QItem,
            QItemMain,
            QPopover,
            QIcon,
            QList,
            QBtn,
            QTab,
            QTabPane,
            QCard,
            QCardMain,
            QCardTitle,
            QInput,
            QSelect,
            QCollapsible,
            VueEditor,
            QSideLink
        },
        data () {
            return {
                chamado: {},
                listaStatus: [],
                listaResponsaveis: [],
                listaTiposChamado: [],
                tipoChamadoId: null,
                responsavelId: null,
                responsavel: {},
                tipoChamado: {},
                statusChamado: {},
                botaoAtivo: true,
                content: '',
                url: 'http://servagilus.dyndns.org:9801/api/'
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
                })

                axios.get('http://servagilus.dyndns.org:9801/api/ResponsavelChamados')
                .then(response => {
                    this.listaResponsaveis = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaResponsaveis.unshift({ label: '', value: null })
                })

                axios.get('http://servagilus.dyndns.org:9801/api/TipoChamados')
                .then(response => {
                    this.listaTiposChamado = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaTiposChamado.unshift({ label: '', value: null })
                })
            })
        },
        methods: {
            trocaResponsavel (value) {
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
            trocaStatus (value, label) {
                let self = this
                if (value !== self.chamado.StatusId) {
                    self.statusChamado = self.listaStatus.find(function (responsaveis) {
                        return (responsaveis.value === value)
                    })
                    axios.put('http://servagilus.dyndns.org:9801/api/TrocaStatusChamado?id=' + this.chamado.Id + '&statusid=' + value)
                    .then(function (response) {
                        if (response.status === 200) {
                            self.chamado.StatusId = value
                            self.chamado.StatusChamado.Nome = label
                            Toast.create.positive({ html: 'Status do Chamado Atualizado' })
                        }
                        else {
                            Toast.create.negative({ html: 'Problemas na Atualização do Status do Chamado' })
                        }
                    })
                }
            },
            enviarMensagem () {
                axios.post(this.url + 'EnvioMensagem?id=' + this.chamado.Id + '&textomensagem=' + this.content)
                .then(response => {
                    this.chamado.MensagensChamado.push({Id: response.data.Id, DataRecebimento: response.data.DataRecebimento, CorpoEmail: this.content, AnexosMensagem: []})
                    this.destruirResposta()
                    this.textoAviso = 'Mensagem enviada'
                    this.aviso = true
                })
            },
            alternarVisibilidadeResposta () {
               this.botaoAtivo = !this.botaoAtivo
            },
            destruirResposta () {
                this.alternarVisibilidadeResposta()
                this.content = ''
            },
            btnresponder () {
                this.botaoAtivo = !this.botaoAtivo
                setTimeout(function () {
                    var top = document.getElementById('editorHtml').offsetTop
                    window.scrollTo(0, top)
                }, 100)
            },
            excluir () {
                axios.delete(this.url + 'Chamados/' + this.chamado.Id).then(response => {
                    if (response.status === 200) {
                        if (this.chamado.Excluido) {
                            this.chamado.Excluido = false
                            Toast.create.positive({ html: 'Chamado ' + this.chamado.Id + ' Restaurado' })
                        }
                        else {
                            this.chamado.Excluido = true
                            Toast.create.positive({ html: 'Chamado ' + this.chamado.Id + ' Excluído' })
                        }
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Exclusão do Chamado ' + this.chamado.Id })
                    }
                })
            }
        }
    }
</script>

<style>
    .mt-1{
        margin-top: 1rem;
    },
    .mb-1{
        margin-bottom: 1rem;
    }
    .pa-1{
        padding: 1rem;
    }

</style>
