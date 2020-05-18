<template>
  <q-page class="flex flex-center">
    <q-table
      style="min-width: 720px"
      title="Žiadosti o predĺženie vypožičky"
      :data="ziadosti"
      :columns="columns"
      :pagination.sync="pagination"
      row-key="id"
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
        { name: 'id', label: '#', align: 'left', field: 'id' },
        { name: 'ziadatel', label: 'Ziadatel', align: 'left', field: 'citatel' },
        { name: 'kniha', align: 'left', label: 'Kniha', field: 'kniha' },
        {
          name: 'datum_range',
          align: 'left',
          label: 'Vypožičaná od-do',
          field: row => {
            return `${row.vypozicana_od} - ${row.vypozicana_do}`
          }
        },
        {
          name: 'pocet_dni',
          align: 'right',
          label: 'predĺženie o (dní)',
          field: row => {
            const d1 = new Date(row.vypozicana_do.split('-')[2], row.vypozicana_do.split('-')[1], row.vypozicana_do.split('-')[0])
            const d2 = new Date(row.predlzenie_do.split('-')[2], row.predlzenie_do.split('-')[1], row.predlzenie_do.split('-')[0])
            return this.dateDiffInDays(d1, d2)
          }
        },
        { name: 'podana_dna', align: 'left', label: 'podaná dňa', field: 'podana_dna' },
        {
          name: 'stav',
          align: 'left',
          label: 'stav',
          field: 'stav',
          format: val => {
            if (val === 0) return 'Čakajúca'
            if (val === 1) return 'Schválená'
            if (val === 2) return 'Zamietnutá'
          }
        }
      ],
      pagination: {
        sortBy: 'id',
        descending: true,
        page: 1,
        rowsPerPage: 10
      },
      ziadosti: []
    }
  },

  mounted () {
    axios
      .get('http://127.0.0.1:8081/api/ziadosti/getAll')
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
    dateDiffInDays (a, b) {
      const _MS_PER_DAY = 1000 * 60 * 60 * 24
      const utc1 = Date.UTC(a.getFullYear(), a.getMonth(), a.getDate())
      const utc2 = Date.UTC(b.getFullYear(), b.getMonth(), b.getDate())
      return Math.floor((utc2 - utc1) / _MS_PER_DAY)
    }
  },

  computed: {
  }
}
</script>

<style scoped>
</style>
