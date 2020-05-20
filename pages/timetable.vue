<template>
  <v-layout>
    <v-flex class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <v-card>
        <v-card-title>
          <v-text-field
            v-model="search"
            append-outer-icon="search"
            label="Enter The Train No."
            single-line
            hide-details
            @click:append-outer="findTimeTable"
            @keyup.enter="findTimeTable"
          ></v-text-field>
        </v-card-title>
        <v-data-table
          :headers="headers"
          :items="desserts"
          :items-per-page="Infinity"
          sort-by="calories"
          class="elevation-1"
          hide-default-footer
        >
          <template v-slot:top>
            <v-toolbar flat color="white">
              <v-toolbar-title>Time Table</v-toolbar-title>
              <v-divider class="mx-4" inset vertical></v-divider>
              <v-spacer></v-spacer>
              <v-dialog v-model="dialog" max-width="500px">
                <!-- <template v-slot:activator="{ on }">
                <v-btn color="primary" dark class="mb-2" v-on="on"
                  >New Item</v-btn
                >
              </template> -->
                <v-card>
                  <v-card-title>
                    <span class="headline">{{ formTitle }}</span>
                  </v-card-title>

                  <v-card-text>
                    <v-container>
                      <v-row>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="editedItem.station"
                            label="Station"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="editedItem.arr_time"
                            label="Arrival Time"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="editedItem.arr_day"
                            label="Arrival Date"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="editedItem.dep_time"
                            label="Departure Time"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="editedItem.dep_day"
                            label="Departure Date"
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
                    <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
          <template v-slot:item.actions="{ item }">
            <v-icon small class="mr-2" @click="editItem(item)">
              mdi-pencil
            </v-icon>
            <v-icon small class="mr-2" @click="deleteItem(item)">
              mdi-delete
            </v-icon>
            <v-icon small class="mr-2" @click="addStation(item.station_idx)">
              mdi-plus-circle
            </v-icon>
          </template>
        </v-data-table>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
export default {
  data: () => ({
    dialog: false,
    headers: [
      {
        text: 'Index',
        align: 'center',
        sortable: false,
        value: 'station_idx'
      },
      {
        text: 'Station',
        align: 'center',
        sortable: false,
        value: 'station'
      },
      {
        text: 'Arr.T',
        align: 'center',
        sortable: false,
        value: 'arr_time'
      },
      {
        text: 'Arr.D',
        align: 'center',
        sortable: false,
        value: 'arr_day'
      },
      {
        text: 'Dep.T',
        align: 'center',
        sortable: false,
        value: 'dep_time'
      },
      {
        text: 'Arr.D',
        align: 'center',
        sortable: false,
        value: 'dep_day'
      },
      { text: 'Actions', align: 'center', value: 'actions', sortable: false }
    ],
    desserts: [],
    editedIndex: -1,
    editedItem: {
      station_idx: 1,
      arr_time: '--:--',
      dep_time: '--:--',
      arr_day: 0,
      dep_day: 0,
      duration: -1,
      station: '',
      code: ''
    },
    defaultItem: {
      station_idx: 1,
      arr_time: '--:--',
      dep_time: '--:--',
      arr_day: 0,
      dep_day: 0,
      duration: -1,
      station: '',
      code: ''
    },
    search: ''
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

  created() {
    this.initialize()
  },

  methods: {
    initialize() {
      this.desserts = []
    },

    editItem(item) {
      const idx = this.editedItem.station_idx
      this.editedIndex = this.desserts.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.editedItem.station_idx = idx
      // console.log(this.editedItem)
      this.dialog = true
      // TO-DO: send to server
    },

    deleteItem(item) {
      const index = this.desserts.indexOf(item)
      confirm('Are you sure you want to delete this item?') &&
        this.desserts.splice(index, 1)
      // TO-DO: send to server
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.desserts[this.editedIndex], this.editedItem)
      } else {
        this.desserts.splice(this.editedItem.station_idx, 0, this.editedItem)
        // TO-DO: send to server
        for (
          let i = this.editedItem.station_idx;
          i < this.desserts.length;
          i++
        ) {
          this.desserts[i].station_idx++
        }
      }
      this.close()
    },
    findTimeTable() {
      const url = '/api/timetable?train_code=' + this.search.toUpperCase()
      fetch(url)
        .then((res) => res.json())
        .then((res) => {
          const { result } = res
          this.desserts = result
        })
        .catch((err) => {
          // eslint-disable-next-line no-console
          console.log(err)
        })
        .finally()
    },
    addStation(index) {
      this.editedItem.station_idx = index
      this.editItem(this.editedItem)

      // console.log(this.desserts)
    }
  }
}
</script>
>
