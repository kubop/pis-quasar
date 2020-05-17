<template>
  <q-page class="flex flex-center">
    <q-table
      style="min-width: 720px"
      title="Žiadosti o predĺženie vypožičky"
      :data="ziadosti"
      :columns="columns"
      row-key="nazov"
      hide-bottom
    />
  </q-page>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Ziadosti',

  data () {
    return {
      columns: [
        {
          name: 'nazov',
          required: true,
          label: 'Ziadatel',
          align: 'left',
          field: 'nazov'
        },
        { name: 'kniha', align: 'right', label: 'Kniha', field: 'kniha', sortable: true },
        { name: 'datum_range', align: 'left', label: 'Vypožičaná od-do', field: 'datum_range', sortable: true },
        { name: 'pocet_dni', align: 'left', label: 'predĺženie o (dní)', field: 'pocet_dni', sortable: true },
        { name: 'podana_dna', align: 'left', label: 'podaná dňa', field: 'podana_dna', sortable: true },
        { name: 'stav', align: 'left', label: 'stav', field: 'stav', sortable: true }
      ],
      ziadosti: []
    }
  },

  mounted () {
    axios
      .get('http://127.0.0.1:8081/ziadosti')
      .then(response => {
        if (response.status === 200) {
          this.ziadosti = response.data
        }
      })
      .catch(error => {
        console.log(error)
        // this.notify(error.message, 'Nastala neočakávaná chyba')
      })
  },

  methods: {
  },

  computed: {
  }
}
</script>

<style scoped>
</style>
