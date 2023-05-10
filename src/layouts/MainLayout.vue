<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="isDrawerOpen = !isDrawerOpen"
        />

        <q-toolbar-title>
          Mpesa/eMola
        </q-toolbar-title>

        <div>e2Payments v1.0.0</div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="isDrawerOpen"
      show-if-above
      :width="300"
      :breakpoint="400"
    >
      <q-scroll-area style="height: calc(100% - 150px); border-right: 1px solid #ddd">
        <q-list padding>
          <q-item to="/" clickable v-ripple>
            <q-item-section avatar>
              <img src="https://user-images.githubusercontent.com/18400142/132318330-b8536515-d0d3-44ba-817e-ee2d269722f6.png"
                 alt=""
                 style="width: 30px;height: 30px"
              >
            </q-item-section>

            <q-item-section>
              Inicio
            </q-item-section>
          </q-item>

        </q-list>
      </q-scroll-area>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>

export default {
  name: 'MainLayout',
  data () {
    return {
      isDrawerOpen: false
    }
  }
}
</script>
