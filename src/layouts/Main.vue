<template>
<q-layout>

  <q-layout-header>
    <q-toolbar>
        <q-btn flat round dense icon="menu" @click="left = !left" class="q-mr-md"/>
        <q-toolbar-title>Admin<span slot="subtitle">wimo</span></q-toolbar-title>
    </q-toolbar>
  </q-layout-header>

  <q-layout-drawer v-model="left" side="left">
    <main-layout-drawer></main-layout-drawer>
  </q-layout-drawer>

  <q-page-container>
    <div class="row full-width justify-center">
      <router-view class="c-container" />
    </div>
  </q-page-container>

  <q-layout-footer>
    <div class="text-center q-pa-md">Copyright (C) 2020, Wimo</div>
  </q-layout-footer>
</q-layout>
</template>

<style>
.c-container {
      width: 95%;
      max-width: 1150px;
}
</style>

<script>
import MainLayoutDrawer from 'components/layouts/main/Drawer.vue'

export default {
  components: {
    MainLayoutDrawer
  },
  computed: {
    left: {
      get () { return this.$store.state.moduleUI.layout.drawerState },
      set (val) { this.$store.commit('moduleUI/updateDrawerState', val) }
    }
  }
}
</script>
