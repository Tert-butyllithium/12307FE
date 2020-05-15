<template>
  <v-layout>
    <v-flex class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <blockquote class="blockquote">
        <!-- &#8220;First, solve the problem. Then, write the code.&#8221;
        <footer>
          <small>
            <em>&mdash;John Johnson</em>
          </small>
        </footer> -->
        <div id="app">
          <v-app id="inspire">
            <v-data-table
              :headers="headers"
              :items="desserts"
              :search="search"
              :loading="dataLoading"
              sort-by="Name"
              class="elevation-1"
            >
              <template v-slot:top>
                <v-toolbar flat color="white">
                  <v-toolbar-title
                    >Common passenger information</v-toolbar-title
                  >
                  <v-spacer></v-spacer>
                  <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                  ></v-text-field>
                  <v-divider class="mx-4" inset vertical></v-divider>
                  <v-spacer></v-spacer>
                  <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on }">
                      <v-btn color="primary" dark class="mb-2" v-on="on"
                        >Add Passenger</v-btn
                      >
                    </template>
                    <v-card>
                      <v-card-title>
                        <span class="headline">{{ formTitle }}</span>
                      </v-card-title>

                      <v-card-text>
                        <v-container>
                          <v-row>
                            <v-col cols="12" sm="6" md="4">
                              <v-text-field
                                v-model="editedItem.name"
                                label="Name"
                              ></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="4">
                              <v-text-field
                                v-model="editedItem.idcard"
                                label="Identity Number"
                              ></v-text-field>
                            </v-col>
                            <v-col cols="12" sm="6" md="4">
                              <v-text-field
                                v-model="editedItem.phone"
                                label="Phone"
                              ></v-text-field>
                            </v-col>
                          </v-row>
                        </v-container>
                      </v-card-text>

                      <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="blue darken-1" text @click="close"
                          >Cancel</v-btn
                        >
                        <v-btn color="blue darken-1" text @click="save"
                          >Save</v-btn
                        >
                      </v-card-actions>
                    </v-card>
                  </v-dialog>
                </v-toolbar>
              </template>
              <template v-slot:item.actions="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)">
                  mdi-pencil
                </v-icon>
                <v-icon small @click="deleteItem(item)">
                  mdi-delete
                </v-icon>
              </template>
              <template v-slot:no-data>
                <v-btn color="primary" @click="initialize">Reset</v-btn>
              </template>
            </v-data-table>
          </v-app>
        </div>
      </blockquote>
    </v-flex>
  </v-layout>
</template>

<script>
import axios from 'axios'
export default {
  inject: ['showLoginDialog'],
  data: () => ({
    search: '',
    dataLoading: true,
    dialog: false,
    headers: [
      {
        text: 'Name',
        align: 'center',
        // sortable: false,
        value: 'name'
      },
      {
        text: 'Identity Number',
        value: 'idcard',
        sortable: false,
        align: 'center'
      },
      { text: 'Phone', value: 'phone', sortable: false, align: 'center' },
      { text: 'Actions', value: 'actions', sortable: false, align: 'center' }
    ],
    desserts: [],
    editedIndex: -1,
    editedItem: {
      name: '',
      idcard: 0,
      phone: 0
    },
    defaultItem: {
      name: '',
      idcard: 0,
      phone: 0
    }
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    }
  },

  watch: {
    dialog(val) {
      val || this.close()
    }
  },

  mounted() {
    this.initialize()
  },

  methods: {
    initialize() {
      // this.desserts = []
      const url = '/api/passenger'
      axios
        .get(url)
        .then((response) => {
          this.desserts = response.data.result
          this.dataLoading = false
        })
        .catch((error) => {
          // console.log(error.response.status)
          if (error.response.status === 401) {
            this.showLoginDialog()
          }
        })
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      const index = this.desserts.indexOf(item)
      if (confirm('Are you sure you want to delete this item?')) {
        const url = '/api/deletepassenger?passenger_id=' + item.id
        axios.get(url).then(() => {
          this.desserts.splice(index, 1)
        })
      }
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      let url
      if (this.editedIndex > -1) {
        Object.assign(this.desserts[this.editedIndex], this.editedItem)
        url = 'api/editpassenger?id=' + this.editedItem.id + '&'
      } else {
        this.desserts.push(this.editedItem)
        url = '/api/addpassenger?'
      }
      url +=
        'name=' +
        this.editedItem.name +
        '&idcard=' +
        this.editedItem.idcard +
        '&phone=' +
        this.editedItem.phone
      axios.get(url).then((response) => {
        this.close()
      })
    }
  }
}
</script>
