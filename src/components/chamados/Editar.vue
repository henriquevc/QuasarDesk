<template>
    <div>
        {{chamado.Id}} | {{chamado.Assunto}} | {{chamado.NomeCliente}} | {{chamado.Email}} | {{chamado.StatusId}}
    </div>
</template>

<script>
    import axios from 'Axios'
    export default {
        data () {
            return {
            chamado: {},
            listaStatus: []
            }
        },
        created () {
             axios.get('http://servagilus.dyndns.org:9801/api/Chamados/' + this.$route.params.id)
             .then(response => {
                this.chamado = response.data
            })
            axios.get('http://servagilus.dyndns.org:9801/api/StatusChamados')
            .then(response => {
                this.listaStatus = response.data.map(function (obj) {
                    return {label: obj.Nome, value: obj.Id}
                })
            })
        }
    }
</script>
