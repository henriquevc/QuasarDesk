<template>
    <div >
        <q-pull-to-refresh :handler="atualizar">
            <div class="shadow-10 pb bg-white">
                <div class="row p3">
                    <q-input class="col-xl-3 col-lg-3 col-md-3 col-xs-12" v-model="buscaId" type="number" float-label="ID" @change="buscar" />
                    <q-input class="col-xl-3 col-lg-3 col-md-3 col-xs-12" v-model="buscaAssunto" type="text" float-label="Assunto" @change="buscar" />
                    <q-select class="col-xl-3 col-lg-3 col-md-3 col-xs-12" ref="responsavel" @focus="selectOptionFocus('responsavel')" autofocusfilter toggle v-model="buscaResponsavel" filter float-label="Responsável" :options="listaResponsaveis" @change="buscar"/>
                    <q-select class="col-xl-3 col-lg-3 col-md-3 col-xs-12" ref="tipoChamado" @focus="selectOptionFocus('tipoChamado')" autofocusfilter toggle v-model="buscaTipoChamado" filter float-label="Tipo do Chamado" :options="listaTiposChamado" @change="buscar"/>
                    <q-select class="col-xl-3 col-lg-3 col-md-3 col-xs-12" ref="orgao" @focus="selectOptionFocus('orgao')" autofocusfilter toggle v-model="buscaOrgaoChamado" filter float-label="Órgão do Chamado" :options="listaOrgaoChamado" @change="buscar"/>
                    <q-select class="col-xl-3 col-lg-3 col-md-3 col-xs-12" ref="status" @focus="selectOptionFocus('status')" autofocusfilter toggle v-model="buscaStatus" filter float-label="Status" :options="listaStatus" @change="buscar"/>
                    <q-datetime class="col-xl-3 col-lg-3 col-md-3 col-xs-12" v-model="buscaDataRecebimento" type="date" format="DD/MM/YYYY" float-label="Data Recebimento" @change="buscar" />
                    <q-select class="col-xl-3 col-lg-3 col-md-3 col-xs-12" ref="cliente" @focus="selectOptionFocus('cliente')" autofocusfilter toggle v-model="buscaCliente" filter float-label="Cliente" :options="listaCliente" @change="buscar"/>
                </div>
            </div>
            <q-card class="mt2 shadow-10 bg-white">
            <div style="overflow:auto;">
                <q-card-main>
                    <table class="q-table striped-odd responsive compact " style="width: 100%; font-size:12px">
                        <thead>
                            <tr>
                                <th :class="coluna.flex" v-for="coluna in columns" :key="coluna">{{coluna.label}}        <q-icon @click="ordenar" size="13px" v-if="coluna.sort" :name="coluna.icon"/></th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(chamado, index) in chamadosFiltrados" :key="chamado">
                                <td><input type="checkbox" v-model="listaChecked" :value="index"></td>
                                <td><router-link class="text-teal bold" :id="chamado.Id" :to="'/editar/'+chamado.Id">{{ chamado.Id }}</router-link></td>
                                <td><div class="elipse">{{chamado.Assunto}}</div></td>
                                <td><q-select class="m" ref="clienteList" @focus="selectOptionFocus('clienteList')" autofocusfilter toggle v-model="chamado.Cliente.Id" filter :options="listaCliente" @change="trocaCliente(chamado.Cliente.Id, chamado.Id)"/></td>
                                <td><q-select class="m" ref="statusList" @focus="selectOptionFocus('statusList')" autofocusfilter toggle v-model="chamado.StatusChamado.Id" filter :options="listaStatus" @change="trocaStatus(chamado.StatusChamado.Id, chamado.Id)"/></td>
                                <td><q-select class="m" ref="responsavelList" @focus="selectOptionFocus('responsavelList')" autofocusfilter toggle v-model="chamado.ResponsavelChamado.Id" filter :options="listaResponsaveis" @change="trocaResponsavel(chamado.ResponsavelChamado.Id, chamado.Id)"/></td>
                                <td><q-select class="m" ref="tipoChamadoList" @focus="selectOptionFocus('tipoChamadoList')" autofocusfilter toggle v-model="chamado.TipoChamado.Id" filter :options="listaTiposChamado" @change="trocaTipoChamado(chamado.TipoChamado.Id, chamado.Id)"/></td>
                                <td><q-select class="m" ref="orgaoList" @focus="selectOptionFocus('orgaoList')" autofocusfilter toggle v-model="chamado.OrgaoChamado.Id" filter :options="listaOrgaoChamado" @change="trocaOrgaoChamado(chamado.OrgaoChamado.Id, chamado.Id)"/></td>
                                <td><div>{{chamado.DataRecebimento.toLocaleDateString()}}</div></td>
                                <td><div v-if="chamado.PrazoFinal">{{chamado.PrazoFinal.toLocaleDateString()}}</div><div v-else>{{chamado.PrazoFinal}}</div></td>
                            </tr>
                        </tbody>
                    </table>   
                    <q-pagination v-model="page" :max="maxpages" @change="paginas"/>
                    <q-btn icon="compare arrows" v-if="listaChecked.length >= 2" color="blue" small @click="mesclaChamados">Mesclar</q-btn>
                    <q-btn icon="delete" v-if="listaChecked.length" color="red" @click="excluir" small>Deletar</q-btn>
                </q-card-main>
                </div>
            </q-card>
        </q-pull-to-refresh>
    </div>
</template>
<script>
    import { Dialog, Toast, QDatetime, QPullToRefresh, QInput, QSelect, QBtn, QPagination, QCard, QCardMain, QCheckbox, QIcon } from 'quasar'
    import axios from 'Axios'

    export default {
        components: { QDatetime, QPullToRefresh, QInput, QSelect, QBtn, QPagination, QCard, QCardMain, QCheckbox, QIcon },
        data () {
            return {
                columns: [
                    { label: '' },
                    { label: 'ID', sort: true, icon: '' },
                    { label: 'Assunto' },
                    { label: 'Remetente' },
                    { label: 'Status' },
                    { label: 'Responsável' },
                    { label: 'Tipo' },
                    { label: 'Órgão' },
                    { label: 'Data de Recebimento' },
                    { label: 'Prazo' }
                ],
                chamados: [],
                chamadosFiltrados: [],
                buscaId: null,
                buscaAssunto: '',
                buscaResponsavel: '',
                buscaDataRecebimento: '',
                buscaTipoChamado: '',
                buscaOrgaoChamado: '',
                buscaStatus: '',
                buscaCliente: '',
                listaStatus: [],
                listaResponsaveis: [],
                listaTiposChamado: [],
                listaOrgaoChamado: [],
                listaCliente: [],
                listaChecked: [],
                anexos: [],
                page: 1,
                maxpages: 1,
                verificaOrdem: true
            }
        },
        beforeCreate () {
            let url = 'http://servagilus.dyndns.org:9801/api/'
            axios.get(url + 'StatusChamados').then(response => {
                this.listaStatus = response.data.map(obj => {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaStatus.unshift({ label: '', value: null })
            })
            axios.get(url + 'ResponsavelChamados').then(response => {
                this.listaResponsaveis = response.data.map(obj => {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaResponsaveis.unshift({ label: '', value: null })
            })
            axios.get(url + 'TipoChamados').then(response => {
                this.listaTiposChamado = response.data.map(obj => {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaTiposChamado.unshift({ label: '', value: null })
            })
            axios.get(url + 'OrgaoChamados').then(response => {
                this.listaOrgaoChamado = response.data.map(obj => {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaOrgaoChamado.unshift({ label: '', value: null })
            })
            axios.get(url + 'Clientes').then(response => {
                this.listaCliente = response.data.map(obj => {
                    return {label: obj.Nome, value: obj.Id}
                })
                this.listaCliente.unshift({ label: '', value: null })
            })
        },
        created () {
            let self = this
            setTimeout(function () {
                let url = 'http://servagilus.dyndns.org:9801/api/'
                let tipoStatus = self.$route.params.tipoStatus
                if (!tipoStatus) {
                    tipoStatus = 'trabalhaveis'
                }
                if (tipoStatus === 'trabalhaveis') {
                    self.columns[1].icon = 'arrow downward'
                    this.verificaOrdem = true
                }
                else {
                    self.columns[1].icon = 'arrow upward'
                    this.verificaOrdem = true
                }
                axios.get(url + 'Chamados?tipoStatus=' + tipoStatus).then(response => {
                    self.chamados = response.data
                    self.chamados.forEach(chamado => {
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
                        if (chamado.PrazoFinal) {
                            chamado.PrazoFinal = new Date(chamado.PrazoFinal)
                        }
                        chamado.DataRecebimento = new Date(chamado.DataRecebimento)
                    })
                    self.maxpages = self.qtdePaginas(self.chamados.length / 20)
                    self.chamadosFiltrados = self.chamados.filter((chamado, index) => {
                        if (index <= 20) {
                            return chamado
                        }
                    })
                })
            }, 500)
        },
        methods: {
            selectOptionFocus (refs) {
                if (typeof this.$refs[refs].$children === 'undefined') {
                    this.$refs[refs][0].$children[0].$children[0].$children[0].focused = true
                    this.$refs[refs][0].$children[0].$children[0].$children[0].focus()
                }
                else {
                    this.$refs[refs].$children[0].$children[0].$children[0].focused = true
                    this.$refs[refs].$children[0].$children[0].$children[0].focus()
                }
            },
            atualizar (done) {
                let url = 'http://servagilus.dyndns.org:9801/api/'
                let tipoStatus = this.$route.params.tipoStatus
                if (!tipoStatus) {
                    tipoStatus = 'trabalhaveis'
                }
                if (tipoStatus === 'trabalhaveis') {
                    this.columns[1].icon = 'arrow downward'
                    this.verificaOrdem = true
                }
                else {
                    this.columns[1].icon = 'arrow upward'
                    this.verificaOrdem = true
                }
                axios.get(url + 'Chamados?tipoStatus=' + tipoStatus).then(response => {
                    this.chamados = response.data
                    this.chamados.forEach(chamado => {
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
                        if (chamado.PrazoFinal) {
                            chamado.PrazoFinal = new Date(chamado.PrazoFinal)
                        }
                        chamado.DataRecebimento = new Date(chamado.DataRecebimento)
                    })
                    this.maxpages = this.qtdePaginas(this.chamados.length / 20)
                    this.chamadosFiltrados = this.chamados.filter((chamado, index) => {
                        if (index <= 20) {
                            return chamado
                        }
                    })
                })
                done()
            },
            ordenar () {
                if (this.$route.params.tipoStatus !== 'trabalhaveis') {
                    if (this.verificaOrdem) {
                        this.verificaOrdem = false
                        this.columns[1].icon = 'arrow downward'
                        this.chamadosFiltrados.sort((a, b) => (a.Id > b.Id) - (a.Id < b.Id))
                    }
                    else {
                        this.verificaOrdem = true
                        this.columns[1].icon = 'arrow upward'
                        this.chamadosFiltrados.sort((a, b) => (a.Id < b.Id) - (a.Id > b.Id))
                    }
                }
                else {
                    if (this.verificaOrdem) {
                        this.verificaOrdem = false
                        this.columns[1].icon = 'arrow upward'
                        this.chamadosFiltrados.sort((a, b) => (a.Id < b.Id) - (a.Id > b.Id))
                    }
                    else {
                        this.verificaOrdem = true
                        this.columns[1].icon = 'arrow downward'
                        this.chamadosFiltrados.sort((a, b) => (a.Id > b.Id) - (a.Id < b.Id))
                    }
                }
            },
            qtdePaginas (n) {
                if (Number(n) === n && n % 1 !== 0) {
                    return Number((n + '').split('.')[0]) + 1
                }
                else {
                    return Number(n)
                }
            },
            paginas (pagina) {
                let inicio = (pagina * 20) - 20
                let fim = (pagina * 20)
                this.chamadosFiltrados = this.chamados.filter((chamado, index) => {
                    return (index >= inicio && index <= fim)
                })
            },
            mesclaChamados () {
                let self = this
                let mescla = []
                this.listaChecked.forEach(index => {
                    mescla.push({label: this.chamadosFiltrados[index].Id + ' - ' + this.chamadosFiltrados[index].Assunto, value: this.chamadosFiltrados[index].Id, index: index})
                })
                Dialog.create({
                    title: 'Mescla',
                    message: 'Selecione o Chamado Principal.',
                    form: {
                        option: {
                            type: 'radio',
                            model: '',
                            inline: true,
                            items: mescla
                        }
                    },
                    buttons: [
                        'Cancel',
                        {
                            label: 'Ok',
                            handler (data) {
                                mescla.forEach(m => {
                                    if (data.option !== m.value) {
                                        self.chamadosFiltrados.forEach((chamado, index) => {
                                            if (m.value === chamado.Id) {
                                                self.chamadosFiltrados.splice(index, 1)
                                                axios.put('http://servagilus.dyndns.org:9801/api/MesclaChamado?idOrigem=' + m.value + '&idDestino=' + data.option).then(function (response) {
                                                    if (response.status === 200) {
                                                        Toast.create.positive({ html: 'Chamado ' + m.value + ' Mesclado Com o Chamado ' + data.option })
                                                    }
                                                    else {
                                                        Toast.create.negative({ html: 'Erro na Mescla' })
                                                    }
                                                })
                                            }
                                        })
                                    }
                                })
                                self.listaChecked = []
                            }
                        }
                    ]
                })
            },
            excluir () {
                let chamadoDeletar = []
                this.listaChecked.forEach(index => {
                    chamadoDeletar.push(this.chamadosFiltrados[index].Id)
                })
                chamadoDeletar.forEach(Id => {
                    this.chamadosFiltrados.forEach((chamado, index) => {
                        if (Id === chamado.Id) {
                            this.chamadosFiltrados.splice(index, 1)
                            axios.delete('http://servagilus.dyndns.org:9801/api/Chamados/' + Id).then(response => {
                                if (response.status === 200) {
                                    Toast.create.positive({ html: 'Chamado ' + Id + ' Excluído' })
                                }
                                else {
                                    Toast.create.negative({ html: 'Problemas na Exclusão do Chamado ' + Id })
                                }
                            })
                        }
                    })
                })
                this.listaChecked = []
            },
            buscar () {
                this.chamadosFiltrados = this.chamados.filter((chamado) => {
                    return (
                        (!this.buscaId || chamado.Id === this.buscaId) &&
                        (!this.buscaAssunto || chamado.Assunto.toUpperCase().indexOf(this.buscaAssunto.toUpperCase()) >= 0) &&
                        (!this.buscaResponsavel || chamado.ResponsavelChamado.Id === this.buscaResponsavel) &&
                        (!this.buscaTipoChamado || chamado.TipoChamado.Id === this.buscaTipoChamado) &&
                        (!this.buscaOrgaoChamado || chamado.OrgaoChamado.Id === this.buscaOrgaoChamado) &&
                        (!this.buscaStatus || chamado.StatusChamado.Id === this.buscaStatus) &&
                        (!this.buscaCliente || chamado.Cliente.Id === this.buscaCliente) &&
                        (!this.buscaDataRecebimento || chamado.DataRecebimento.substring(0, 10) === this.buscaDataRecebimento.substring(0, 10))
                    )
                })
                if (!this.buscaId && !this.buscaAssunto && !this.buscaResponsavel && !this.buscaTipoChamado && !this.buscaOrgaoChamado && !this.buscaStatus && !this.buscaCliente && !this.buscaDataRecebimento) {
                    this.maxpages = this.qtdePaginas(this.chamados.length / 20)
                    this.chamadosFiltrados = this.chamados.filter((chamado, index) => {
                        if (index <= 20) {
                            return chamado
                        }
                    })
                }
                else {
                    this.maxpages = 1
                }
            },
            trocaResponsavel (value, id) {
                let self = this
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaResponsavelChamado?id=' + id + '&responsavelid=' + value).then(function (response) {
                    if (response.status === 200) {
                        self.chamadosFiltrados.forEach(chamado => {
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
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaTipoChamado?id=' + id + '&tipoChamadoId=' + value).then(function (response) {
                    if (response.status === 200) {
                        self.chamadosFiltrados.forEach(chamado => {
                            if (chamado.Id === id) {
                                if (value === 1) {
                                    chamado.PrazoFinal = self.diasUteis(chamado.DataRecebimento, 3)
                                }
                                else {
                                    chamado.PrazoFinal = ''
                                }
                            }
                        })
                        Toast.create.positive({ html: 'Tipo de Chamado Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Tipo de Chamado' })
                    }
                })
            },
            trocaOrgaoChamado (value, id) {
                let self = this
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaOrgaoChamado?id=' + id + '&orgaoChamadoId=' + value).then(function (response) {
                    if (response.status === 200) {
                        self.chamadosFiltrados.forEach(chamado => {
                            if (chamado.Id === id) {
                                chamado.OrgaoChamado.Id = value
                            }
                        })
                        Toast.create.positive({ html: 'Órgão de Chamado Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Órgão de Chamado' })
                    }
                })
            },
            trocaStatus (value, id) {
                axios.put('http://servagilus.dyndns.org:9801/api/TrocaStatusChamado?id=' + id + '&statusid=' + value).then(function (response) {
                    if (response.status === 200) {
                        this.chamadosFiltrados.forEach(chamado => {
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
            },
            trocaCliente (ClienteId, id) {
                let url = 'http://servagilus.dyndns.org:9801/api/'
                axios.put(url + 'TrocaCliente?id=' + id + '&clienteId=' + ClienteId).then(response => {
                    if (response.status === 200) {
                        this.chamadosFiltrados.forEach(chamado => {
                            if (chamado.Id === id) {
                                chamado.Cliente.Id = ClienteId
                            }
                        })
                        Toast.create.positive({ html: 'Cliente Atualizado' })
                    }
                    else {
                        Toast.create.negative({ html: 'Problemas na Atualização do Cliente' })
                    }
                })
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
            }
        }
    }
</script>

<style lang="stylus">
    .p3
        padding 0 .75rem
    .mt2
        margin-top 2rem
    .m
        margin-top 0rem
        margin-bottom 0rem
        font-size 12px
    .pb
        padding-bottom 1.8rem
    .bold
        font-weight bold
    .tr
        backgroud-color white
    .elipse
        max-width 200px
        overflow hidden
        text-overflow ellipsis
        white-space nowrap
</style>
