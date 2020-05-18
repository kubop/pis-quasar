<template>
  <q-page class="flex flex-center">

    <div class="row" style="max-width: 1300px">
      <q-table
        class="col-8"
        style="min-width: 720px"
        title="Moje výpožičky"
        :data="vypozicky_knihy"
        :columns="columns"
        row-key="id"
        selection="single"
        :selected.sync="selected"
        :pagination.sync="pagination"
      />

      <div class="aside col-4 q-pa-md">
        <h5 class="q-ma-none q-mb-md">Predĺžiť výpožičku</h5>
        <p><b>1. Zvoľte výpožičku</b>, ktorú si prajete predĺžiť</p>
        <p><b>2. Zvoľte lehotu</b> do kedy si plánujete výpožičku predĺžiť:</p>

        <q-input
          filled
          v-model="date"
          mask="date"
          :rules="['date']"
          :error="date_error"
          :error-message="date_message"
        >
          <template v-slot:append>
            <q-icon name="event" class="cursor-pointer">
              <q-popup-proxy ref="qDateProxy" transition-show="scale" transition-hide="scale">
                <q-date
                  v-model="date"
                  :locale="myLocale"
                  @input="() => $refs.qDateProxy.hide()"
                />
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>

        <p><b>3. Uvedte dôvod</b> žiadosti o predĺženie:</p>
        <q-input
          v-model="dovod_citatel"
          filled
          type="textarea"
          placeholder="Sem napíšte váš dôvod"
          :error="dovod_citatel_error"
          :error-message="dovod_citatel_message"
        />

        <div class="flex flex-center q-pt-md">
          <q-btn @click="onSubmit" class="text-weight-regular" color="primary" label="Predĺžiť výpožičku" />
        </div>
      </div>
    </div>

  </q-page>
</template>

<script>
import axios from 'axios'
// import * as soap from 'soap'

export default {
  name: 'Vypozicky',

  data () {
    return {
      dovod_citatel: '',
      dovod_citatel_error: false,
      dovod_citatel_message: '',

      date: '2020-01-01',
      date_error: false,
      date_message: '',

      vypozicky: [],
      knihy: [],

      selected: [],
      columns: [
        {
          name: 'nazov',
          required: true,
          label: 'Názov',
          align: 'left',
          field: row => {
            if (row.kniha) {
              return row.kniha.nazov
            }
          }
        },
        {
          name: 'datum_od',
          align: 'right',
          label: 'od',
          field: 'datum_od',
          format: val => val.split('-').join('.')
        },
        {
          name: 'datum_do',
          align: 'left',
          label: 'do',
          field: 'datum_do',
          format: val => val.split('-').join('.'),
          sort: (a, b) => {
            const d1 = new Date(a.split('-')[2], a.split('-')[1], a.split('-')[0])
            const d2 = new Date(b.split('-')[2], b.split('-')[1], b.split('-')[0])
            return d1 - d2
          }
        }
      ],
      pagination: {
        sortBy: 'datum_do',
        descending: true
      },

      myLocale: {
        /* starting with Sunday */
        days: 'Nedeľa_Pondelok_Utorok_Streda_Štvrtok_Piatok_Sobota'.split('_'),
        daysShort: 'Ned_Pon_Ut_Str_Štv_Pia_Sob'.split('_'),
        months: 'Január_Február_Marec_Apríl_Máj_Jún_Júl_August_September_Október_November_December'.split('_'),
        monthsShort: 'Jan_Feb_Mar_Apr_Máj_Jún_Júl_Aug_Sep_Okt_Nov_Dec'.split('_'),
        firstDayOfWeek: 1
      }
    }
  },

  mounted () {
    this.date = new Date().toISOString().slice(0, 10)
    axios
      .post('http://127.0.0.1:8081/api/vypozicky/getByCitatelID', {
        citatel: 1
      }).then(response => {
        if (response && response.status === 200 && response.data && response.data.length >= 1) {
          this.vypozicky = response.data
          axios
            .post('http://127.0.0.1:8081/api/knihy/getByIDs', {
              ids: this.knihy_ids
            }).then(response => {
              if (response && response.status === 200 && response.data && response.data.length >= 1) {
                this.knihy = response.data
              }
            }).catch(error => {
              console.log(error)
            })
        }
      }).catch(error => {
        console.log(error)
      })
    /*
    var url = 'http://localhost:8081/wsdl?WSDL'
    var args = { email: 'value' }
    soap.createClient(url, function (err, client) {
      if (err) {
        console.log(err)
        return
      }
      client.validateEmail(args, function (err, result) {
        if (err) {
          console.log(err)
          return
        }
        console.log(result)
      })
    })
    */
  },

  methods: {
    onSubmit () {
      if (this.selected.length !== 1) {
        this.$q.dialog({
          title: 'Chyba',
          message: 'Nevybrali ste žiadnu výpožičku na predĺženie'
        })
        return
      }

      // Dôvod predĺženie nemôže byť prázdny
      if (!this.dovod_citatel) {
        this.dovod_citatel_error = true
        this.dovod_citatel_message = 'Dôvod predĺženia nemôže byť prázdny'
      } else {
        this.dovod_citatel_error = false
        this.dovod_citatel_message = ''
      }

      // Dátum nemôže byť v minulosti
      const d1 = new Date(this.date).setHours(0, 0, 0, 0)
      const d2 = new Date(this.convertToDate(this.selected[0].datum_do)).setHours(0, 0, 0, 0)

      if (d1 <= d2) {
        this.date_error = true
        this.date_message = 'Dátum musí byť väčší ako dátum konca výpožičky'
      } else {
        this.date_error = false
        this.date_message = ''
      }

      // Ak je nejaky error, nepokračujeme
      if (this.date_error || this.dovod_citatel_error) {
        return
      }

      axios
        .post('http://127.0.0.1:8081/api/vypozicky/predlzenieVypozicky', this.ziadost)
        .then(response => {
          if (response.status === 200) {
            if (response.data.error) {
              this.alert('Žiadosť zamietnutá', 'Predĺženie zvolenej výpožičky nie je možné z dôvodu: ' + response.data.error)
            } else {
              this.$q.dialog({
                title: 'Povtrdenie žiadosti',
                message: 'Prosím overte si dané údaje:<br/>' +
                          'Kniha: {this.selected.kniha.nazov}<br/>'.replace('{this.selected.kniha.nazov}', this.selected[0].kniha.nazov) +
                          'Predĺženie do: {this.date}<br/>'.replace('{this.date}', this.date) +
                          'Dôvod: {this.dovod_citatel}'.replace('{this.dovod_citatel}', this.dovod_citatel),
                html: true,
                ok: {
                  push: true,
                  label: 'Potvrdiť'
                },
                cancel: {
                  push: true,
                  label: 'Zatvoriť',
                  color: 'negative'
                }
              }).onOk(() => {
                axios
                  .post('http://127.0.0.1:8081/api/ziadosti/vytvoritNovu', this.ziadost)
                  .then(response => {
                    if (response.status === 200) {
                      this.alert(
                        'Úspešné podanie žiadosti',
                        'Vaša žiadosť o predĺženie výpožičky bola odoslaná na spracovanie. O jej priebehu budete informovaný emailom, prípadne SMS.'
                      )
                    }
                  })
                  .catch(error => {
                    console.log(error)
                    this.alert('Chyba', 'Nepodarilo sa vytvoriť novú žiadosť')
                  })
              })
            }
          }
        })
        .catch(error => {
          console.log(error)
          this.alert('Chyba', 'Nepodarilo sa overiť udaje na serveri')
        })
    },

    alert (title, message) {
      this.$q.dialog({
        title: title,
        message: message
      })
    },

    convertToDate (date) {
      const d = date.split('-')
      return `${d[2]}/${d[1]}/${d[0]}`
    }
  },

  computed: {
    ziadost: {
      get () {
        return {
          vypozicka: { ...this.selected[0] },
          predlzenie_do: this.computedDate,
          dovod_predlzenia: this.dovod_citatel
        }
      }
    },
    computedDate: {
      get () {
        const d = this.date.split('/')
        return `${d[2]}-${d[1]}-${d[0]}`
      }
    },
    knihy_ids: {
      get () {
        return this.vypozicky.map(v => {
          return { id: v.kniha_id }
        })
      }
    },
    vypozicky_knihy: {
      get () {
        return this.vypozicky.map(v => {
          return {
            ...v,
            kniha: this.knihy.find(k => k.id === v.kniha_id)
          }
        })
      }
    }
  }
}
</script>

<style scoped>
div .aside {
  border-radius: 4px;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.2), 0 2px 2px rgba(0, 0, 0, 0.14), 0 3px 1px -2px rgba(0, 0, 0, 0.12);
}
</style>
