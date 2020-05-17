<template>
  <v-layout>
    <v-flex class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <blockquote class="blockquote">
        <div id="app">
          <v-app id="inspire">
            <v-data-table
              :headers="headers"
              :items="desserts"
              :search="search"
              :loading="dataLoading"
              sort-by="Order ID"
              class="elevation-1"
            >
              <template v-slot:top>
                <v-toolbar flat color="white">
                  <v-toolbar-title>Orderlist</v-toolbar-title>
                  <v-spacer></v-spacer>
                  <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                  ></v-text-field>
                  <!-- <v-divider class="mx-4" inset vertical></v-divider> -->
                  <!-- <v-spacer></v-spacer> -->
                  <v-dialog v-model="dialog" max-width="500px">
                    <v-card>
                      <v-card-title>
                        <span class="headline"> Details </span>
                      </v-card-title>

                      <v-card-text> Order ID: {{}} </v-card-text>

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
                <v-icon small class="mr-2" @click="moreInfo(item)">
                  mdi-dots-horizontal
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
        text: 'Order ID',
        align: 'center',
        // sortable: false,
        value: 'order_id'
      },
      {
        text: 'Passenger',
        value: 'passenger_name',
        sortable: false,
        align: 'center'
      },
      {
        text: 'Train ID',
        value: 'train_num',
        sortable: false,
        align: 'center'
      },
      {
        text: 'Departure Station',
        value: 'dep_station',
        sortable: false,
        align: 'center'
      },
      {
        text: 'Arrivial Station',
        value: 'arr_station',
        sortable: false,
        align: 'center'
      },
      {
        text: 'Departure Time',
        value: 'dep_time',
        sortable: true,
        align: 'center'
      },
      {
        text: 'Price',
        value: 'seat_price',
        sortable: false,
        align: 'center'
      },
      { text: 'Actions', value: 'actions', sortable: false, align: 'center' }
    ],
    desserts: [],
    editedIndex: -1,
    editedItem: {
      order_id: 0,
      passenger_name: null,
      train_num: null,
      dep_station: null,
      arr_station: null,
      seat_no: null,
      seat_price: null,
      seat_type: null,
      purchase_time: null,
      dep_time: null,
      arr_time: null
    },
    defaultItem: {
      order_id: 0,
      passenger_name: null,
      train_num: null,
      dep_station: null,
      arr_station: null,
      seat_no: null,
      seat_price: null,
      seat_type: null,
      purchase_time: null,
      dep_time: null,
      arr_time: null
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
      const url = '/api/orderlist'
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

    moreInfo(item) {
      this.dialog = true
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
        'order_id=' +
        this.editedItem.order_id +
        '&passenger_name=' +
        this.editedItem.passenger_name +
        '&train_num=' +
        this.editedItem.train_num
      axios.get(url).then((response) => {
        this.close()
      })
    }
  }
}
</script>
