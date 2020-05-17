<template>
  <v-layout>
    <v-flex class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <blockquote class="blockquote">
        <v-card>
          <v-card-title class="headline blue lighten-2">
            Search Train Tickets
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-autocomplete
                v-model="model"
                :items="items"
                :loading="isLoading"
                :search-input.sync="deptSearch"
                hide-no-data
                hide-selected
                item-text="station"
                item-value="API"
                label="Departure Station"
                placeholder="Start typing to Search"
                prepend-icon="mdi-airplane-takeoff"
                return-object
              ></v-autocomplete>

              <v-autocomplete
                v-model="model2"
                :items="items"
                :loading="isLoading"
                :search-input.sync="termiSearch"
                hide-no-data
                hide-selected
                item-text="station"
                item-value="API"
                label="Arrival Station"
                placeholder="Start typing to Search"
                prepend-icon="mdi-airplane-landing"
                return-object
              ></v-autocomplete>

              <v-menu
                ref="menu"
                v-model="menu"
                :close-on-content-click="false"
                :return-value.sync="date"
                transition="scale-transition"
                offset-y
                min-width="290px"
              >
                <template v-slot:activator="{ on }">
                  <v-text-field
                    v-model="date"
                    label="Date"
                    prepend-icon="event"
                    readonly
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="date"
                  no-title
                  scrollable
                  :min="new Date().toISOString().substr(0, 10)"
                >
                  <v-spacer></v-spacer>
                  <v-btn text color="primary" @click="menu = false"
                    >Cancel</v-btn
                  >
                  <v-btn text color="primary" @click="$refs.menu.save(date)"
                    >OK</v-btn
                  >
                </v-date-picker>
              </v-menu>
            </v-row>
          </v-card-text>
          <!-- <v-divider></v-divider> -->
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              :disabled="!model && !model2"
              @click="model = model2 = searched = null"
            >
              Clear
              <v-icon right>mdi-close-circle</v-icon>
            </v-btn>
            <v-btn :disabled="!model || !model2" @click="search">
              Search
              <v-icon right>mdi-magnify</v-icon>
            </v-btn>
          </v-card-actions>
          <v-expand-transition>
            <!-- <v-list v-if="searched" class="blue lighten-3"></v-list> -->
            <v-data-table
              v-if="searched"
              :headers="tableHeaders"
              :items="lines"
              :expanded.sync="expanded"
              :loading="dataTableLoading"
              item-key="id"
              show-expand
              class="elevation-1"
            >
              <template v-slot:item.actions="{ item }">
                <v-icon small class="mr-2" @click="buyItem(item)">
                  mdi-currency-usd
                </v-icon>
              </template>
            </v-data-table>
          </v-expand-transition>
          <v-dialog v-model="dialog" max-width="500px" persistent>
            <v-card>
              <v-card-title>
                <span class="headline">Buy a Ticket</span>
              </v-card-title>
              <v-spacer></v-spacer>
              <v-card-text>
                <v-container>
                  <v-radio-group v-model="radioGroup">
                    <v-radio
                      v-for="n in tobuy_types"
                      :key="n.type_id"
                      :label="
                        `${n.type_name} ￥${n.price}  |  Remaining Tickets: ${n.ticket}`
                      "
                      :disabled="!n.ticket"
                      :value="n.type_id"
                    ></v-radio>
                  </v-radio-group>

                  <v-select
                    v-model="selectedPassengers"
                    :items="passengerItems"
                    :item-value="getIdid"
                    :item-text="getIdcard"
                    attach
                    chips
                    label="Select Passenger"
                    multiple
                  ></v-select>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="dialog = false"
                  >Cancel</v-btn
                >
                <v-btn
                  color="blue darken-1"
                  text
                  :disabled="!radioGroup"
                  @click="buybuy"
                  >Buy</v-btn
                >
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-card>
      </blockquote>
    </v-flex>
  </v-layout>
</template>
<script>
export default {
  data() {
    return {
      descriptionLimit: 60,
      entries: [],
      isLoading: false,
      model: null,
      model2: null,
      // search: null
      deptSearch: null,
      termiSearch: null,
      searched: null,
      date: new Date().toISOString().substr(0, 10),
      menu: false,
      expanded: [],
      dataTableLoading: true,
      lines: [],
      tableHeaders: [
        {
          text: 'Train ID',
          align: 'center',
          sortable: false,
          value: 'train_num'
        },
        {
          text: 'Departure Station',
          value: 'dep_station',
          align: 'center',
          sortable: false
        },
        {
          text: 'Arrival Station',
          value: 'arr_station',
          align: 'center',
          sortable: false
        },
        { text: 'Departure Time', value: 'dep_time', align: 'center' },
        { text: 'Total Time', value: 'total_time_in_hour', align: 'center' },
        { text: 'Arrival Time', value: 'arr_time', align: 'center' },
        { text: 'Actions', value: 'actions', sortable: false, align: 'center' }
      ],
      exactQuery: 0,
      tableCount: 0,
      dialog: false,
      tobuy: {},
      tobuy_types: [],
      tobuy_types_length: 0,
      radioGroup: 0,
      passengerItems: [],
      selectedPassengers: []
    }
  },
  computed: {
    items() {
      return this.entries.map((entry) => {
        const station =
          entry.station.length > this.descriptionLimit
            ? entry.station.slice(0, this.descriptionLimit) + '...'
            : entry.station

        return Object.assign({}, entry, { station })
      })
    }
  },

  watch: {
    deptSearch(val) {},
    termiSearch(val) {}
  },
  mounted() {
    fetch('/stations.json')
      .then((res) => res.json())
      .then((res) => {
        // eslint-disable-next-line camelcase
        const { result_cnt, result } = res
        // eslint-disable-next-line camelcase
        this.count = result_cnt
        this.entries = result
        // console.log(result.length)
      })
      .catch((err) => {
        // eslint-disable-next-line no-console
        console.log(err)
      })
      .finally(() => (this.isLoading = false))
    // get passengers info
    fetch('/api/passenger')
      .then((res) => res.json())
      .then((res) => {
        // eslint-disable-next-line camelcase
        const { result } = res
        // eslint-disable-next-line camelcase
        this.passengerItems = result
        // console.log(result.length)
      })
      .catch((err) => {
        // eslint-disable-next-line no-console
        console.log(err)
      })
      .finally()
  },
  methods: {
    search() {
      // console.log(this.model.code)
      // console.log(this.model2.code)
      const now = new Date()
      this.searched = true
      const url =
        '/api/query?dep=' +
        this.model.code +
        '&arr=' +
        this.model2.code +
        '&date=' +
        this.date.split('-').join('') +
        '&exact=' +
        this.exactQuery
      fetch(url)
        .then((res) => res.json())
        .then((res) => {
          // eslint-disable-next-line camelcase
          const { result_cnt, result } = res
          if (this.date === now.toISOString().substr(0, 10)) {
            this.lines = []
            for (let i = 0; i < result.length; i++) {
              if (
                this.time_cmp(
                  now.getHours(),
                  now.getMinutes(),
                  result[i].dep_time,
                  30
                )
              ) {
                this.lines.push(result[i])
              }
            }
            this.tableCount = this.lines.length
          } else {
            // eslint-disable-next-line camelcase
            this.tableCount = result_cnt
            this.lines = result
          }
          for (let i = 0; i < this.lines.length; i++) {
            this.process(this.lines[i])
            this.lines[i].id = i
          }
          // console.log(result.length)
        })
        .catch((err) => {
          // eslint-disable-next-line no-console
          console.log(err)
        })
        .finally(() => (this.dataTableLoading = false))
    },
    more_then_day(depTime, totalTime) {
      const dd = depTime.split(':')
      const h = Number(dd[0])
      const m = Number(dd[1])
      return Math.floor((h * 60 + m + totalTime) / 1440)
    },
    process(item) {
      const hour = Math.floor(item.total_time / 60)
      if (hour) {
        item.total_time_in_hour =
          hour +
          ' hour' +
          (hour === 1 ? ' ' : 's ') +
          (item.total_time % 60) +
          ' minutes'
      } else {
        item.total_time_in_hour = (item.total_time % 60) + ' minutes'
      }
      const moreThanDay = this.more_then_day(item.dep_time, item.total_time)
      // console.log(moreThanDay)
      if (moreThanDay) {
        item.arr_time +=
          ' (+' + moreThanDay + ' day' + (moreThanDay === 1 ? '' : 's') + ')'
      }
    },
    time_cmp(ha, ma, b, t) {
      // const aa = a.split(':')
      const bb = b.split(':')
      // const ha = Number(aa[0])
      // const ma = Number(aa[1])
      const hb = Number(bb[0])
      const mb = Number(bb[1])
      return ha * 60 + ma + t < hb * 60 + mb
    },
    buyItem(item) {
      this.tobuy.arr_idx = item.arr_idx
      this.tobuy.dep_idx = item.dep_idx
      this.tobuy.date = this.date.split('-').join('')
      this.tobuy.train_id = item.train_id
      // console.log(item)

      const url =
        '/api/price?train_id=' +
        item.train_id +
        '&dep_idx=' +
        item.dep_idx +
        '&arr_idx=' +
        item.arr_idx +
        '&date=' +
        this.tobuy.date

      fetch(url)
        .then((res) => res.json())
        .then((res) => {
          // eslint-disable-next-line camelcase
          const { result_cnt, result } = res
          // eslint-disable-next-line camelcase
          this.tobuy_types = result
          // eslint-disable-next-line camelcase
          this.tobuy_types_length = result_cnt
          // console.log(result.length)
        })
        .catch((err) => {
          // eslint-disable-next-line no-console
          console.log(err)
        })
        .finally()
      this.dialog = true
    },
    buybuy() {
      // console.log(this.radioGroup)
      // console.log(this.tobuy_types[Number(this.radioGroup) - 1])
      // const realTypeId = this.tobuy_types[Number(this.radioGroup) - 1].type_id
      const url =
        '/api/purchase?train_id=' +
        this.tobuy.train_id +
        '&dep_idx=' +
        this.tobuy.dep_idx +
        '&arr_idx=' +
        this.tobuy.arr_idx +
        '&date=' +
        this.tobuy.date +
        '&type_id=' +
        this.radioGroup +
        '&passenger_id='
      const info = []
      // alert(this.selectedPassengers)
      // console.log(this.radioGroup + ' ' + realTypeId)
      for (let x = 0; x < this.selectedPassengers.length; x++) {
        const i = this.selectedPassengers[x]
        fetch(url + i)
          .then((res) => res.json())
          .then((res) => {
            // eslint-disable-next-line camelcase
            const { errcode, errmsg, seat_no, order_id } = res
            if (errcode) {
              alert(errmsg)
              return
            }
            info.push({
              person: this.passengerItems[i],
              order_id,
              seat_no
            })
          })
          .catch((err) => {
            // eslint-disable-next-line no-console
            console.log(err)
          })
          .finally()
      }
      this.dialog = false
      this.selectedPassengers = []
      // console.log(info)
    },
    getIdid(item) {
      // console.log(item.id)
      // console.log(this.selectedPassengers)
      return item.id
    },
    getIdcard(item) {
      return (
        item.name +
        ' ' +
        item.idcard.substr(0, 5) +
        '...' +
        item.idcard.substr(15, 3)
      )
    }
  }
}
</script>
