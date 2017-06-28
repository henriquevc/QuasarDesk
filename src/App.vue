<template>
  <!-- Don't drop "q-app" class -->
  <div id="q-app">
    <q-layout ref="layout" view="lHh Lpr fff" :left-class="{'lime-4': true}">

      <q-toolbar slot="header">
        <q-btn flat @click="$refs.layout.toggleLeft()">
          <q-icon name="menu" />
        </q-btn>

        <q-toolbar-title>
          <q-icon name="headset_mic" /> <span>Agilus Desk</span>
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
            <q-item-side icon="work" />
            <q-item-main label="Abertos" />
          </q-side-link>
          <q-side-link item to="/listar/fechados">
            <q-item-side icon="highlight_off" />
            <q-item-main label="Fechados"/>
          </q-side-link>
          <q-side-link item to="/listar/todos">
            <q-item-side icon="list" />
            <q-item-main label="Todos" />
          </q-side-link>
        </q-list>
      </div>

      <!--
        Replace following <div> with
        <router-view /> component
        if using subRoutes
      -->
      <div class="layout-padding">
        <!-- your content -->
        <router-view :key="$route.path"/>
      </div>
    </q-layout>
  </div>
</template>

<script>
import {
  QLayout,
  QToolbar,
  QToolbarTitle,
  QSearch,
  QTabs,
  QRouteTab,
  QBtn,
  QIcon,
  QItemSide,
  QItemMain,
  QSideLink,
  QListHeader,
  QScrollArea,
  QList,
  QInput,
  QAjaxBar
} from 'quasar'
/*
 * Root component
 */
export default {
  components: {
    QLayout,
    QToolbar,
    QToolbarTitle,
    QSearch,
    QTabs,
    QRouteTab,
    QBtn,
    QIcon,
    QItemSide,
    QItemMain,
    QSideLink,
    QListHeader,
    QScrollArea,
    QList,
    QInput,
    QAjaxBar
  },
  data () {
    return {
    }
  }
}
</script>

<style></style>
