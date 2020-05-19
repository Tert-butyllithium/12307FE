<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8 md6>
      <div class="text-center">
        <logo />
        <vuetify-logo />
      </div>
      <v-card>
        <v-card-title class="headline">
          Welcome to the 12307 Railway Ticketing System
        </v-card-title>
        <v-card-text>
          <p>
            In our 12307 railway ticketing system, we provide a light system for
            your management about railway tickets.
          </p>
          <p>
            Following functions are provided in this system.
          </p>
          <ul>
            <li>Query the timetable of given train</li>
            <li>Purchase train tickets</li>
            <li>Recommendation for transfer routes</li>
            <li>Management of orders and common passengers</li>
          </ul>
          <p></p>
          <p>
            Thank you for experiencing our online tickets reserving system.
          </p>
          <hr class="my-3" />
          The UI is based on
          <a href="https://vuetifyjs.com" target="_blank"
            >Vuetify + Nuxt.js template</a
          >.
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn color="primary" nuxt to="/purchase">Purchase</v-btn>
          <v-btn color="primary" nuxt to="/order">Order List</v-btn>
          <!-- <v-btn color="primary" @click="loginout"> Log In/ Log Out</v-btn> -->
        </v-card-actions>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import Logo from '~/components/Logo.vue'
import VuetifyLogo from '~/components/VuetifyLogo.vue'

export default {
  components: {
    Logo,
    VuetifyLogo
  },
  data: () => ({
    account: '',
    password: '',
    dialog: false,
    backend: 'http://sh.wtd2.top:81'
    // login_success: window.localStorage.getItem('token') !== undefined
  })
}
</script>
