<template>
  <q-page class="flex flex-center">

    <q-form
      @submit="onSubmit"
      class="q-gutter-md"
    >
      <h2 class="q-mb-sm">Prihlásenie</h2>
      <q-input
        filled
        v-model="email"
        label="Email"
        placeholder="Zadajte váš email"
        type="email"
        lazy-rules
        :rules="[ val => val && val.length > 0 || 'Zadajte email']"
      />

      <q-input
        v-model="heslo"
        filled
        :type="isPwd ? 'password' : 'text'"
        label="Heslo"
        lazy-rules
        :rules="[ val => val && val.length > 0 || 'Zadajte heslo']"
      >
        <template v-slot:append>
          <q-icon
            :name="isPwd ? 'visibility_off' : 'visibility'"
            class="cursor-pointer"
            @click="isPwd = !isPwd"
          />
        </template>
      </q-input>

      <div>
        <a href="#/prihlasenie/obnovenie-hesla">
          Zabudli ste heslo?
        </a>
      </div>

      <div class="flex flex-center">
        <q-btn class="full-width text-weight-regular" label="Prihlásiť" type="submit" color="primary"/>
      </div>
    </q-form>
  </q-page>
</template>

<script>
import * as soap from 'soap'

export default {
  name: 'Prihlasenie',

  data () {
    return {
      email: '',
      heslo: '',
      isPwd: true
    }
  },

  methods: {
    onSubmit () {
      const url = 'http://localhost:8081/wsdl/Auth?WSDL'
      const args = {
        email: this.email,
        heslo: this.heslo
      }
      soap.createClient(url, (err, client) => {
        if (err) return console.log(err)

        client.login(args, (err, result) => {
          if (err) return console.log(err)

          if (result.success) {
            this.$q.dialog({
              title: 'Úspešné prihlásenie',
              message: 'Prihlásili ste sa (akože :-))'
            })
          } else {
            this.$q.dialog({
              title: 'Chyba',
              message: 'Zadaný email alebo heslo je nesprávne'
            })
          }
        })
      })
    }
  },

  computed: {
  }
}
</script>

<style scoped>
</style>
