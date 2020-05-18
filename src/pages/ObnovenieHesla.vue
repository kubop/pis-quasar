<template>
  <q-page class="flex flex-center">

    <q-form
      @submit="onSubmit"
      class="q-gutter-md"
    >
      <h2 class="q-mb-sm">Obnovenie hesla</h2>
      <q-input
        filled
        v-model="email"
        label="Email"
        lazy-rules
        :rules="[ val => val && val.length > 0 || 'Zadajte email']"
      />

      <div>
        <q-btn class="q-mb-md full-width text-weight-regular" label="obnoviť" type="submit" color="primary"/>
        <q-btn to="/prihlasenie" class="full-width text-weight-regular" label="späť na prihlásenie" color="standart" text-color="black"/>
      </div>
    </q-form>
  </q-page>
</template>

<script>
import axios from 'axios'

export default {
  name: 'ObnovenieHesla',

  data () {
    return {
      email: null,
      gotError: false,
      errorMessage: null
    }
  },

  methods: {

    onSubmit () {
      if (!this.email) {
        return
      }

      axios
        .post('http://127.0.0.1:8081/api/obnovenie-hesla/validaciaEmailu', this.emailJson)
        .then(response => {
          if (response.status === 200) {
            if (!response.data.valid) {
              this.alert('Chyba', 'Zadali ste zlý formát emailovej adresy')
            } else {
              axios
                .post('http://127.0.0.1:8081/api/obnovenie-hesla/odoslanieEmailu', this.emailJson)
                .then(response => {
                  if (response.status === 200) {
                    if (response.data.error) {
                      this.alert('Chyba', response.data.error)
                    } else {
                      this.alert('Inštrukcie odoslané', 'Na Vami zadaný email bola zaslaná správa s ďalšími inštrukciami')
                    }
                  }
                })
                .catch(error => {
                  console.log(error)
                  // _self.notify(error.message, 'Nastala neočakávaná chyba')
                })
            }
          }
        })
        .catch(error => {
          console.log(error)
          this.notify(error.message, 'Nastala neočakávaná chyba')
        })
    },

    alert (title, message) {
      this.$q.dialog({
        title: title,
        message: message
      }).onOk(() => {
        // console.log('OK')
      }).onCancel(() => {
        // console.log('Cancel')
      }).onDismiss(() => {
        // console.log('I am triggered on both OK and Cancel')
      })
    }
  },

  computed: {
    emailJson: {
      get () { return { email: this.email } }
    }
  }
}
</script>

<style scoped>
</style>
