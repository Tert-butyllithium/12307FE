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
              <template v-slot:expanded-item="{ headers, item }">
                <td :colspan="headers.length">More info about {{ item.id }}</td>
              </template>
            </v-data-table>
          </v-expand-transition>
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
      lines: [
        {
          train_id: 1234,
          dep_idx: 1,
          arr_idx: 20,
          train_num: 'Z42',
          dep_station: '乌鲁木齐',
          arr_station: '上海',
          dep_time: '19:31',
          arr_time: '11:59',
          total_time: 2428,
          ticket_yz: 0,
          price_yz: 385.5,
          ticket_rz: 0,
          price_rz: 615.5,
          ticket_sw: -1,
          price_sw: -1,
          ticket_yw: 0,
          price_yw: 695.5,
          ticket_rw: 0,
          price_rw: 1075.5
        }
      ],
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
        { text: '', value: 'data-table-expand' }
      ],
      exactQuery: 0,
      tableCount: 0
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
    }
  }
}
</script>
