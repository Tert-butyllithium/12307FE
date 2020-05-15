<template>
  <v-app dark>
    <v-dialog v-model="login_dialog" persistent max-width="500px">
      <v-card>
        <v-toolbar dark color="primary">
          <v-toolbar-title></v-toolbar-title>
          <v-spacer>Login</v-spacer>
          <v-tooltip bottom></v-tooltip>
        </v-toolbar>
        <v-card-text>
          <v-form>
            <v-text-field
              v-model="account"
              prepend-icon="person"
              name="login"
              label="Account"
              type="text"
            ></v-text-field>
            <v-text-field
              id="password"
              v-model="password"
              prepend-icon="lock"
              name="password"
              label="Password"
              type="password"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="normal" @click="login_dialog = false">Cancel</v-btn>
          <v-btn color="primary" @click="login">Login</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-navigation-drawer
      v-model="drawer"
      :mini-variant="miniVariant"
      :clipped="clipped"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-btn icon @click.stop="miniVariant = !miniVariant">
        <v-icon>mdi-{{ `chevron-${miniVariant ? 'right' : 'left'}` }}</v-icon>
      </v-btn>
      <v-btn icon @click.stop="clipped = !clipped">
        <v-icon>mdi-application</v-icon>
      </v-btn>
      <v-btn icon @click.stop="fixed = !fixed">
        <v-icon>mdi-minus</v-icon>
      </v-btn>
      <v-toolbar-title v-text="title" />
      <v-spacer />
      <v-btn v-show="islogin" icon @click="loginout">
        <v-icon>
          mdi-account
        </v-icon>
        <!-- <div v-else>
          Login
        </div> -->
      </v-btn>
      <v-btn v-show="!islogin" icon @click="loginout">
        Login
      </v-btn>
    </v-app-bar>
    <v-content>
      <v-container>
        <nuxt />
      </v-container>
    </v-content>
    <!-- <v-navigation-drawer v-model="rightDrawer" :right="right" temporary fixed>
      <v-list>
        <v-list-item @click.native="right = !right">
          <v-list-item-action>
            <v-icon light>
              mdi-repeat
            </v-icon>
          </v-list-item-action>
          <v-list-item-title>Switch drawer (click me)</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer> -->
    <v-footer :fixed="fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import axios from 'axios'
export default {
  name: 'App',
  data() {
    return {
      islogin: true,
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        },
        {
          icon: 'mdi-train',
          title: 'Ticket Purchase',
          to: '/purchase'
        },
        {
          icon: 'mdi-cart',
          title: 'Order Management',
          to: '/order'
        },
        {
          icon: 'mdi-account-multiple',
          title: 'Passenger management',
          to: '/passengers'
        }
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: '12307 Railway ticketing system',
      account: '',
      password: '',
      login_dialog: false
    }
  },
  provide() {
    return {
      showLoginDialog: this.showLoginDialog
    }
  },
  mounted() {
    const url = '/api/status'
    axios
      .get(url)
      .then((response) => {
        this.islogin = response.data.errcode === 1
      })
      .catch(() => {
        this.islogin = false
      })
    // console.log('awsl')
  },
  methods: {
    loginfailed() {
      alert('Wrong user name or password')
      this.account = ''
      this.password = ''
    },
    login() {
      const url =
        '/api/login?username=' + this.account + '&password=' + this.password
      axios.get(url).then((response) => {
        if (response.status === 200) {
          localStorage.setItem('token', response.data.token)
          localStorage.setItem('username', this.account)
          this.account = ''
          this.password = ''
          // this.login_success = true
          this.login_dialog = false
          alert('Login successful')
          window.location.reload(false)
        } else {
          this.loginfailed()
        }
      })
    },
    loginout() {
      if (
        localStorage.getItem('token') === undefined ||
        localStorage.getItem('token') == null
      ) {
        this.login_dialog = true
      } else if (confirm('Are you sure to logout')) {
        // console.log(localStorage.getItem('token'))
        const url = '/api/logout'
        // axios.post(url, {
        //   account: localStorage.getItem('account')
        // })
        axios.get(url)
        localStorage.clear()
        alert('Logout successful')
        window.location.reload(false)
      }
    },
    logined() {
      const url = '/api/status'
      axios.get(url).then((response) => {
        return response.data.errcode === 1
      })

      return false
    },
    showLoginDialog() {
      this.login_dialog = true
    }
  }
}
</script>
