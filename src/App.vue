<template>
  <!-- Don't drop "q-app" class -->
  <div id="q-app">
    <q-layout ref="layout" view="lHh Lpr fff" :left-class="{'bg-secondary': true, 'text-white': true}">

      <q-toolbar slot="header">
        <q-btn flat @click="$refs.layout.toggleLeft()">
          <q-icon name="menu" />
        </q-btn>

        <q-toolbar-title>
          <router-link class="text-white bold" to='/listar/trabalhaveis'> <q-icon name="headset_mic" /> <span>Agilus Desk</span> </router-link> 
          <div slot="subtitle">Sistema para gerenciamento de chamados</div>
        </q-toolbar-title>
      </q-toolbar>

      <div slot="left">
        <!--
          Use <q-side-link> component 
          instead of <q-item> for 
          internal vue-router navigation
        -->
        <q-list no-border link inset-delimiter>
          <q-side-link item to="/listar/trabalhaveis">
            <q-item-side icon="work" color="white" />
            <q-item-main label="Abertos" @click="$refs.layout.toggleLeft()"/>
          </q-side-link>
          <q-side-link item to="/listar/fechados">
            <q-item-side icon="highlight_off" color="white" />
            <q-item-main label="Fechados" @click="$refs.layout.toggleLeft()"/>
          </q-side-link>
          <q-side-link item to="/controle/bases">
            <q-item-side icon="date range" color="white" />
            <q-item-main label="Controle" @click="$refs.layout.toggleLeft()"/>
          </q-side-link>
          <q-side-link item to="/listar/excluidos">
            <q-item-side icon="delete" color="white" />
            <q-item-main label="Excluidos" @click="$refs.layout.toggleLeft()"/>   
          </q-side-link>                             
          <q-side-link item to="/listar/todos">
            <q-item-side icon="list" color="white" />
            <q-item-main label="Todos" @click="$refs.layout.toggleLeft()"/>
          </q-side-link>
        </q-list>
      </div>

      <!--
        Replace following <div> with
        <router-view /> component
        if using subRoutes
      -->
      <div class="layout-padding bg-light">
        <!-- your content -->
        <router-view :key="$route.path"/>
      </div>
    </q-layout>
  </div>
</template>

<script>
import {QLayout, QToolbar, QToolbarTitle, QSearch, QTabs, QRouteTab, QBtn, QIcon, QItemSide, QItemMain, QSideLink, QListHeader, QScrollArea, QList, QInput, QAjaxBar} from 'quasar'
/*
 * Root component
 */
export default {
  components: { QLayout, QToolbar, QToolbarTitle, QSearch, QTabs, QRouteTab, QBtn, QIcon, QItemSide, QItemMain, QSideLink, QListHeader, QScrollArea, QList, QInput, QAjaxBar },
  data () {
    return {
    }
  }
}
</script>

<style lang="stylus">
    .layout-padding
      padding 1rem 2rem
      min-height 100vh
      height 100%
</style>
