<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="bg-dark text-white" height-hint="98">
      <q-toolbar>
        <q-toolbar-title>
          Knižnica
        </q-toolbar-title>
      </q-toolbar>

      <q-tabs align="left">
        <q-route-tab to="/" label="Domov" />
        <q-route-tab to="/vypozicky" label="Moje vypozicky" />
        <q-route-tab to="/ziadosti" label="ziadosti" />
        <q-route-tab to="/prihlasenie" label="prihlasenie" />
      </q-tabs>

    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>

  </q-layout>
</template>

<script>

</script>
