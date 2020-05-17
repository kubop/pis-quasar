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
      />

      <q-input
        filled
        v-model="heslo"
        label="Heslo"
      />

      <div>
        <a href="#/obnovenie-hesla">
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
export default {
  name: 'Prihlasenie',

  data () {
    return {
      email: null,
      heslo: null
    }
  },

  methods: {
    onSubmit () {
      console.log('here')
    }
  },

  computed: {
  }
}
</script>

<style scoped>
</style>
