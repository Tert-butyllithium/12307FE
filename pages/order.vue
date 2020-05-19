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
                  <v-toolbar-title>Order List</v-toolbar-title>
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

                      <v-card-text>
                        Order ID: {{ editedItem.order_id }} <br />
                        Passenger: {{ editedItem.passenger_name }} |
                        {{ editedItem.passenger_idcard }} <br />
                        Train ID: {{ editedItem.train_num }} <br />
                        Interval: {{ editedItem.dep_station }} ->
                        {{ editedItem.arr_station }} <br />
                        Interval Time: {{ editedItem.dep_time }} ->
                        {{ editedItem.arr_time }} <br />
                        Seat: {{ editedItem.seat_no }} |
                        {{ editedItem.seat_type }} <br />
                        Price: {{ editedItem.seat_price }} <br />
                        Purchase Time: {{ editedItem.purchase_time }} <br />
                      </v-card-text>

                      <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn
                          color="error"
                          depressed
                          :disabled="
                            new Date(editedItem.dep_time) - new Date() <
                              allowRefund
                          "
                          @click="refund"
                          >Refund</v-btn
                        >
                        <!-- <v-col cols="1"></v-col> -->
                        <v-btn color="blue" depressed @click="close"
                          >Close</v-btn
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
        text: 'Train No',
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
        text: 'Arrival Station',
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
      arr_time: null,
      passenger_idcard: null
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
      arr_time: null,
      passenger_idcard: null
    },
    allowRefund: 25 * 60000
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
      this.editedItem = item
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    canRefund() {
      const now = new Date()
      if (
        this.editedItem.dep_time.substr(0, 10) ===
        now.toISOString().substr(0, 10)
      )
        return new Date(this.editedItem.dep_time) - now > this.allowRefund
      if (
        this.editedItem.dep_time.substr(0, 10) < now.toISOString().substr(0, 10)
      )
        return false
      return true
    },
    refund() {
      const index = this.desserts.indexOf(this.editedItem)
      if (!this.canRefund()) {
        alert(
          'Error: You cannot refund the ticket because it is less than 25 minutes before the departure time'
        )
        return
      }
      if (confirm('Are you sure you want to refund this ticket?')) {
        const url = '/api/refund?order_id=' + this.editedItem.order_id
        axios.get(url).then(() => {
          this.desserts.splice(index, 1)
        })
      }
      this.dialog = false
    }
  }
}
</script>
