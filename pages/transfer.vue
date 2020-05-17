<template>
  <v-layout>
    <v-flex class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <blockquote class="blockquote">
        <v-card :loading="!prepared">
          <v-card-title class="headline green lighten-2">
            Transfer Search
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-autocomplete
                v-if="prepared"
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
                v-if="prepared"
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
              <v-select
                v-if="prepared"
                v-model="exactQuery"
                :items="option"
                item-text="text"
                item-value="value"
                label="Advanced query options"
                prepend-icon="mdi-apple-keyboard-option"
              >
              </v-select>
            </v-row>
          </v-card-text>
          <!-- <v-divider></v-divider> -->
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              :disabled="dataTableLoading || (!model && !model2)"
              @click="model = model2 = searched = null"
            >
              Clear
              <v-icon right>mdi-close-circle</v-icon>
            </v-btn>
            <v-btn
              :disabled="dataTableLoading || !model || !model2"
              @click="search"
            >
              Search
              <v-icon right>mdi-magnify</v-icon>
            </v-btn>
          </v-card-actions>
          <v-expand-transition>
            <v-data-table
              v-if="searched"
              :headers="tableHeaders"
              :items="lines"
              :loading="dataTableLoading"
              item-key="id"
              class="elevation-1"
            >
              <template v-slot:item.actions="{ item }">
                <v-icon small class="mr-2" @click="buyItem(item)">
                  mdi-currency-usd
                </v-icon>
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
      prepared: false,
      isLoading: false,
      model: null,
      model2: null,
      // search: null
      deptSearch: null,
      termiSearch: null,
      searched: null,
      date: new Date().toISOString().substr(0, 10),
      menu: false,
      // expanded: [],
      dataTableLoading: false,
      lines: [],
      durTimeSort: (a, b) => {
        const getHM = (a) => {
          const aa = a.split(' ')
          if (aa.length < 4) {
            return Number(aa[0])
          } else {
            return 60 * Number(aa[0]) + Number(aa[2])
          }
        }
        return getHM(a) - getHM(b)
      },
      tableHeaders: [
        {
          text: 'First Train',
          align: 'center',
          sortable: false,
          value: 'first_train'
        },
        {
          text: 'Second Train',
          align: 'center',
          sortable: false,
          value: 'second_train'
        },
        {
          text: 'Departure Station',
          value: 'dep_station',
          align: 'center',
          sortable: false
        },
        {
          text: 'Transfer station',
          value: 'via_station',
          align: 'center',
          sortable: false
        },
        {
          text: 'Arrival station',
          value: 'arr_station',
          align: 'center',
          sortable: false
        },
        {
          text: 'First Departure Time',
          value: 'first_dep_time',
          align: 'center',
          sortable: false
        },
        {
          text: 'First Arrvial Time',
          value: 'first_arr_time',
          align: 'center',
          sortable: false
        },
        {
          text: 'Second Departure Time',
          value: 'second_dep_time',
          align: 'center',
          sortable: false
        },
        {
          text: 'First Arrvial Time',
          value: 'second_arr_time',
          align: 'center',
          sortable: false
        },
        {
          text: 'Total Time',
          value: 'total_time_in_hour',
          align: 'center',
          sort: this.durTimeSort
        },
        {
          text: 'Transfer Time',
          value: 'transfer_time_in_hour',
          align: 'center',
          sort: this.durTimeSort
        }
      ],
      exactQuery: 0,
      tableCount: 0,
      dialog: false,
      tobuy: {},
      tobuy_types: [],
      tobuy_types_length: 0,
      radioGroup: 0,
      passengerItems: [],
      selectedPassengers: [],
      sucessTicktInfoDialog: false,
      sucessPassengers: [],
      option: [
        {
          text: 'Same City',
          value: 0
        },
        {
          text: 'Precise Station',
          value: 1
        },
        {
          text: 'City Central Station (experimental)',
          value: 2
        }
      ]
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
      .finally((this.prepared = true))
  },
  methods: {
    search() {
      // console.log(this.model.code)
      // console.log(this.model2.code)
      this.dataTableLoading = true
      this.lines = []
      const now = new Date()
      this.searched = true
      const url =
        '/api/transfer?dep=' +
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
                  result[i].first_dep_time,
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
      const hourTransfer = Math.floor(item.transfer_time / 60)
      const mm = item.transfer_time % 60
      if (hourTransfer) {
        item.transfer_time_in_hour =
          hourTransfer +
          ' hour' +
          (hour === 1 ? ' ' : 's ') +
          mm +
          ' minute' +
          (mm === 1 ? '' : 's')
      } else {
        item.transfer_time_in_hour = mm + ' minute' + (mm === 1 ? '' : 's')
      }
      const moreThanDay = this.more_then_day(
        item.first_dep_time,
        item.total_time
      )
      if (moreThanDay) {
        item.second_arr_time += ' (+' + moreThanDay + ')'
      }
    },
    time_cmp(ha, ma, b, t) {
      const bb = b.split(':')
      const hb = Number(bb[0])
      const mb = Number(bb[1])
      return ha * 60 + ma + t < hb * 60 + mb
    }
  }
}
</script>
