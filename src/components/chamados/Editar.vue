<template>
    <div>
        <q-tabs align="justify" class="mb-1 shadow-10" style="margin-bottom: 16px;" v-if="chamado.StatusChamado" v-model="chamado.StatusChamado.Nome">
            <q-tab slot="title" :name="status.label" :icon="status.icone" :disable="status.disable" v-for="status in listaStatus" :key="status" @select="trocaStatus(status.value, status.label)">{{status.label}}</q-tab>
        </q-tabs>
        <div class="row">
            <div class="col-lg-4 col-md-12">
                <q-card class="pa-1 shadow-10 bg-white">
                    <q-input stack-label="Assunto" readonly v-model="chamado.Assunto"/>
                    <q-input stack-label="E-mail" readonly v-model="chamado.Email"/>
                    <q-select ref="cliente" @focus="selectOptionFocus('cliente')" autofocusfilter toggle v-model="chamado.ClienteId" filter float-label="Cliente" :options="listaCliente" @change="trocaCliente"/>
                    <q-select ref="responsavel" @focus="selectOptionFocus('responsavel')" autofocusfilter toggle v-model="chamado.ResponsavelId" filter float-label="Responsável" :options="listaResponsaveis" @change="trocaResponsavel"/>
                    <q-select ref="tipoChamado" @focus="selectOptionFocus('tipoChamado')" autofocusfilter toggle v-model="chamado.TipoChamadoId" filter float-label="Tipo Chamado" :options="listaTiposChamado" @change="trocaTipoChamado"/>
                    <q-select ref="orgao" @focus="selectOptionFocus('orgao')" autofocusfilter toggle v-model="chamado.OrgaoId" filter float-label="Órgão Chamado" :options="listaOrgaoChamado" @change="trocaOrgaoChamado"/>
                    <q-datetime v-model="chamado.PrazoFinal" type="date" format="DD/MM/YYYY 00:00:00" float-label="Prazo Final" @change="atualizaPrazoFinal"/>
                    <q-input stack-label="Data Finalização" :disable="true" v-model="chamado.DataFinalizacao"/>
                </q-card>  
            </div>
            <div>
                <q-fab direction="up" class="fixed" style="right: 2rem; bottom: .5rem; z-index: 1" color="primary" icon="keyboard_arrow_up" active-icon="keyboard_arrow_down" push>
                    <q-fab-action round color="red" v-show="!chamado.Excluido" @click="excluir" icon="delete">
                        <q-tooltip anchor="center left" self="center right" :offset="[20, 0]">Excluir</q-tooltip>
                    </q-fab-action>
                    <q-fab-action round color="tertiary" v-show="chamado.Excluido" @click="excluir" icon="undo">
                        <q-tooltip anchor="center left" self="center right" :offset="[20, 0]">Restaurar</q-tooltip>
                    </q-fab-action>
                    <q-fab-action round color="blue" @click="mesclaChamado(chamado.Id)" icon="compare arrows">
                        <q-tooltip anchor="center left" self="center right" :offset="[20, 0]">Mesclar</q-tooltip>
                    </q-fab-action>
                    <q-fab-action round color="info" v-show="botaoAtivo" @click="btnAviso" icon="announcement">
                        <q-tooltip anchor="center left" self="center right" :offset="[20, 0]">Aviso</q-tooltip>
                    </q-fab-action>
                    <q-fab-action round color="info" v-show="botaoAtivo" @click="btnresponder" icon="reply">
                        <q-tooltip anchor="center left" self="center right" :offset="[20, 0]">Responder</q-tooltip>
                    </q-fab-action>
                </q-fab>                               
            </div>                             
            <div class="col-lg-8 col-md-12">
                <q-card v-for="mensagem in chamado.MensagensChamado" :key="mensagem" class="bg-white shadow-10" style="margin-bottom: 1rem;">
                    <q-card-title class="bg-secondary text-white">
                        #{{mensagem.Id}} - {{ mensagem.DataRecebimento }}
                        <q-icon color="white" slot="right" name="attach file" v-if="mensagem.AnexosMensagem.length">
                                <q-popover ref="popover">
                                    <q-list link class="no-border">
                                        <q-item link tag="a" exact :href="url+'AnexosMensagens/'+anexo.Id" v-for="anexo in mensagem.AnexosMensagem" :key="anexo">
                                            <q-item-main :label="anexo.NomeAnexo"/>
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
                            <vue-editor id="" v-model="content" v-show="!botaoAtivo" id="editorHtml" style=" height: 285px;"> </vue-editor>
                        </div>
                    </div><!--col-12-->
                </q-card>          
                <div class="mb-1 mt-1">
                    <q-btn flat class="bg-positive text-white" v-show="!botaoAtivo" @click="enviarMensagem()">Enviar <q-icon right name="send"/></q-btn>
                    <q-btn flat class="bg-negative text-white" v-show="!botaoAtivo" @click="destruirResposta()">Cancelar <q-icon right name="block"/></q-btn>
                    <q-btn flat class="bg-blue text-white" v-show="!botaoAtivo" @click="arquivoSelecionado">Anexo <q-icon right name="attach file"/></q-btn>
                    <q-icon v-show="!botaoAtivo" color="black" size="32px"  name="attachment" v-if="nomeAnexos.length">                    
                        <q-popover ref="popover">
                            <q-list class="no-border">
                                <q-item v-for="anexo in nomeAnexos" :key="anexo">
                                    <q-item-main :label="anexo"/>
                                </q-item>
                            </q-list>
                        </q-popover>
                    </q-icon>
                </div><!--col-12-->                         
            </div>            
        </div>
    </div>
</template>

<script>
    import axios from 'Axios'
    import { VueEditor } from 'vue2-editor'
    import { Toast, Dialog, QFab, QFabAction, QTabs, QItem, QTooltip, QItemMain, QPopover, QIcon, QList, QBtn, QTab, QTabPane, QCard, QCardMain, QCardTitle, QInput, QSelect, QCollapsible, QSideLink, QDatetime } from 'quasar'

    export default {
        components: { QFab, QFabAction, QTabs, QItem, QTooltip, QItemMain, QPopover, QIcon, QList, QBtn, QTab, QTabPane, QCard, QCardMain, QCardTitle, QInput, QSelect, QCollapsible, VueEditor, QSideLink, QDatetime },
        data () {
            return {
                chamado: {},
                listaStatus: [],
                listaResponsaveis: [],
                listaTiposChamado: [],
                listaOrgaoChamado: [],
                listaCliente: [],
                tipoChamadoId: null,
                responsavelId: null,
                responsavel: {},
                tipoChamado: {},
                statusChamado: {},
                botaoAtivo: true,
                aviso: false,
                content: '',
                url: 'http://servagilus.dyndns.org:9801/api/',
                input: null,
                anexos: null,
                nomeAnexos: []
            }
        },
        beforeCreate () {
            let url = 'http://servagilus.dyndns.org:9801/api/'
            axios.get(url + 'ResponsavelChamados').then(response => {
                this.listaResponsaveis = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaResponsaveis.unshift({ label: '', value: null })
            })

            axios.get(url + 'TipoChamados').then(response => {
                this.listaTiposChamado = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaTiposChamado.unshift({ label: '', value: null })
            })

            axios.get(url + 'OrgaoChamados').then(response => {
                this.listaOrgaoChamado = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaOrgaoChamado.unshift({ label: '', value: null })
            })

            axios.get(url + 'Clientes').then(response => {
                this.listaCliente = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaCliente.unshift({ label: '', value: null })
            })
        },
        created () {
             axios.get('http://servagilus.dyndns.org:9801/api/Chamados/' + this.$route.params.id).then(response => {
                this.chamado = response.data
                if (!this.chamado.PrazoFinal) {
                    this.chamado.PrazoFinal = ''
                }
                if (!this.chamado.DataFinalizacao) {
                    this.chamado.DataFinalizacao = ''
                }
                else {
                    this.chamado.DataFinalizacao = (new Date(this.chamado.DataFinalizacao).toLocaleString())
                }
                axios.get('http://servagilus.dyndns.org:9801/api/StatusChamados').then(response => {
                    this.listaStatus = response.data.map(function (obj) {
                        return {label: obj.Nome, value: obj.Id}
                    })
                    this.listaStatus[0].icone = 'message'
                    this.listaStatus[1].icone = 'build'
                    this.listaStatus[2].icone = 'done'
                    this.listaStatus[3].icone = 'stop'
                    this.listaStatus[4].icone = 'alarm'
                    this.listaStatus[4].disable = true
                })
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
            atualizaPrazoFinal () {
                let self = this
                axios({
                    url: self.url + 'Chamados/' + self.chamado.Id,
                    method: 'put',
                    data: self.chamado
                }).then(response => {
                    if (response.status === 200) {
                        Toast.create.positive({ html: 'Prazo Final Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Prazo Final' })
                    }
                })
            },
            arquivoSelecionado (arquivo) {
                let self = this
                this.fileselected = false
                this.input = document.createElement('input')
                this.input.type = 'file'
                this.input.multiple = true
                this.input.click()
                this.input.addEventListener('change', handleFiles, false)
                function handleFiles () {
                    self.nomeAnexos = []
                    for (let i = 0; i < this.files.length; i++) {
                        self.nomeAnexos.push(this.files[i].name)
                    }
                }
            },
            trocaCliente (value) {
                axios({
                    url: this.url + 'Chamados/' + this.chamado.Id,
                    method: 'put',
                    data: this.chamado
                }).then(response => {
                    if (response.status === 204) {
                        Toast.create.positive({ html: 'Cliente Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Cliente' })
                    }
                })
            },
            trocaResponsavel (value) {
                axios.put(this.url + 'TrocaResponsavelChamado?id=' + this.chamado.Id + '&responsavelid=' + value)
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
                axios.put(this.url + 'TrocaTipoChamado?id=' + this.chamado.Id + '&tipoChamadoId=' + value).then(function (response) {
                    if (response.status === 200) {
                        if (value === 1) {
                            self.chamado.PrazoFinal = self.diasUteis(self.chamado.DataRecebimento, 3)
                        }
                        else {
                            self.chamado.PrazoFinal = ''
                        }
                        Toast.create.positive({ html: 'Tipo de Chamado Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Tipo de Chamado' })
                    }
                })
            },
            trocaOrgaoChamado (value) {
                axios.put(this.url + 'TrocaOrgaoChamado?id=' + this.chamado.Id + '&orgaoChamadoId=' + value)
                .then(function (response) {
                    if (response.status === 200) {
                        Toast.create.positive({ html: 'Órgão Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Órgão' })
                    }
                })
            },
            trocaStatus (value, label) {
                let self = this
                if (value !== self.chamado.StatusId) {
                    self.statusChamado = self.listaStatus.find(function (responsaveis) {
                        return (responsaveis.value === value)
                    })
                    axios.put(this.url + 'TrocaStatusChamado?id=' + this.chamado.Id + '&statusid=' + value)
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
            diasUteis (data, dias) {
                let novaData = new Date(data)
                if (novaData.getHours() < 12) {
                    dias--
                }
                let cont = 1
                while (cont <= dias) {
                    novaData.setDate(novaData.getDate() + 1)
                    if (novaData.getDay() === 6 || novaData.getDay() === 0) {
                        dias++
                    }
                    cont++
                }
                return novaData
            },
            enviarMensagem () {
                let self = this
                let achou = false
                let anexosMaiores = {tamanho: '', nome: ''}
                if (this.input) {
                    for (let i = 0; i <= this.input.files.length; i++) {
                        if (this.input.files[i] && this.input.files[i].size >= 20971520) {
                            achou = true
                            anexosMaiores.tamanho += (this.input.files[i].size * 1024) * 1024
                            anexosMaiores.nome += this.input.files[i].name + '<br>'
                        }
                    }
                }
                if (achou) {
                    Dialog.create({
                        title: 'Anexos',
                        message: 'O(s) Arquivo(s)' + '<br>' + '<br>' + anexosMaiores.nome + '<br>' + 'Ultrapassa(m) 20Mb',
                        buttons: [
                            'Ok'
                        ]
                    })
                }
                else {
                    if (this.input) {
                        this.anexos = this.tranformaAnexos(this.input.files)
                        setTimeout(function () {
                            axios({
                                url: self.url + 'EnvioMensagem?id=' + self.chamado.Id + '&textomensagem=' + self.content + '&aviso=' + self.aviso,
                                method: 'post',
                                data: self.anexos
                            }).then(response => {
                                let anexosInseridos = []
                                response.data.idAnexos.forEach((id, index) => {
                                    anexosInseridos.push({Id: id, NomeAnexo: self.anexos.AnexoNome[index]})
                                })
                                self.chamado.MensagensChamado.push({Id: response.data.Id, CorpoEmail: self.content, DataRecebimento: response.data.DataRecebimento, AnexosMensagem: anexosInseridos})
                                self.destruirResposta()
                                self.textoAviso = 'Mensagem enviada'
                                self.nomeAnexos = []
                            })
                        }, 100)
                    }
                    else {
                        setTimeout(function () {
                            axios({
                                url: self.url + 'EnvioMensagem?id=' + self.chamado.Id + '&textomensagem=' + self.content + '&aviso=' + self.aviso,
                                method: 'post',
                                data: {Anexo: [], AnexoNome: []}
                            }).then(response => {
                                self.chamado.MensagensChamado.push({Id: response.data.Id, CorpoEmail: self.content, DataRecebimento: response.data.DataRecebimento, AnexosMensagem: []})
                                self.destruirResposta()
                                self.textoAviso = 'Mensagem enviada'
                            })
                        }, 100)
                    }
                }
            },
            tranformaAnexos (arquivos) {
                let retorno = {Anexo: [], AnexoNome: []}
                for (let i = 0; i <= arquivos.length; i++) {
                    let reader = new FileReader()
                    reader.onload = function (e) {
                        retorno.Anexo.push(e.target.result)
                        retorno.AnexoNome.push(arquivos[i].name)
                    }
                    if (arquivos[i]) {
                        reader.readAsDataURL(arquivos[i])
                    }
                    reader = null
                }
                return retorno
            },
            alternarVisibilidadeResposta () {
                this.botaoAtivo = !this.botaoAtivo
                this.aviso = false
            },
            destruirResposta () {
                this.alternarVisibilidadeResposta()
                this.content = ''
                this.anexos = null
                this.nomeAnexos = []
            },
            btnresponder () {
                this.botaoAtivo = !this.botaoAtivo
                setTimeout(function () {
                    let top = document.getElementById('editorHtml').offsetTop
                    window.scrollTo(0, top)
                }, 100)
            },
            btnAviso () {
                this.botaoAtivo = !this.botaoAtivo
                this.aviso = !this.aviso
                setTimeout(function () {
                    let top = document.getElementById('editorHtml').offsetTop
                    window.scrollTo(0, top)
                }, 100)
            },
            excluir () {
                axios.delete(this.url + 'Chamados/' + this.chamado.Id).then(response => {
                    if (response.status === 200) {
                        if (this.chamado.Excluido) {
                            this.chamado.Excluido = false
                            Toast.create.positive({ html: 'Chamado ' + this.chamado.Id + ' Restaurado da Lixeira' })
                        }
                        else {
                            this.chamado.Excluido = true
                            Toast.create.positive({ html: 'Chamado ' + this.chamado.Id + ' Enviado para a Lixeira' })
                        }
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Exclusão do Chamado ' + this.chamado.Id })
                    }
                })
            },
            mesclaChamado (idDestino) {
                let self = this
                Dialog.create({
                    title: 'Mescla de Chamados',
                    message: 'Digite o código do chamado para a mescla',
                      form: {
                        name: {
                        type: 'number',
                        label: 'Código',
                        model: ''
                        }
                    },
                    buttons: [
                        'Cancel',
                        {
                            label: 'Ok',
                            handler (idOrigem) {
                                axios.get('http://servagilus.dyndns.org:9801/api/Chamados/' + idOrigem.name).then(function (chamadoOrigem) {
                                    axios.put('http://servagilus.dyndns.org:9801/api/MesclaChamado?idOrigem=' + idOrigem.name + '&idDestino=' + idDestino).then(function (response) {
                                        if (response.status === 200) {
                                            chamadoOrigem.data.MensagensChamado.forEach(function (mensagem) {
                                                self.chamado.MensagensChamado.push({ Id: mensagem.Id, DataRecebimento: mensagem.DataRecebimento, CorpoEmail: mensagem.CorpoEmail, AnexosMensagem: mensagem.AnexosMensagem })
                                            })
                                            Toast.create.positive({ html: 'Chamado ' + idOrigem.name + ' Mesclado Com o Chamado ' + idDestino })
                                        }
                                        else {
                                            Toast.create.negative({ html: 'Erro na Mescla' })
                                        }
                                    })
                                })
                            }
                        }
                    ]
                })
            }
        }
    }
</script>

<style>
    .mt-1{
        margin-top: 1rem;
    }
    .mb-1{
        margin-bottom: 1rem;
    }
    .pa-1{
        padding: 1rem;
    }

</style>
