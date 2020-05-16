<template>
  <v-layout>
    <v-flex class="text-center">
      <img src="/v.png" alt="Vuetify.js" class="mb-5" />
      <blockquote class="blockquote">
        <v-card color="blue lighten-2">
          <v-card-title class="headline blue lighten-3">
            Search for Public APIs
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-autocomplete
                v-model="model"
                :items="items"
                :loading="isLoading"
                :search-input.sync="deptSearch"
                color="white"
                hide-no-data
                hide-selected
                item-text="station"
                item-value="API"
                label="Public APIs"
                placeholder="Start typing to Search"
                prepend-icon="mdi-airplane-takeoff"
                return-object
              ></v-autocomplete>

              <v-autocomplete
                v-model="model2"
                :items="items"
                :loading="isLoading"
                :search-input.sync="termiSearch"
                color="white"
                hide-no-data
                hide-selected
                item-text="station"
                item-value="API"
                label="Public APIs"
                placeholder="Start typing to Search"
                prepend-icon="mdi-airplane-landing"
                return-object
              ></v-autocomplete>
            </v-row>
          </v-card-text>
          <!-- <v-divider></v-divider> -->
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn :disabled="!model && !model2" @click="model = model2 = null">
              Clear
              <v-icon right>mdi-close-circle</v-icon>
            </v-btn>
            <v-btn :disabled="!model || !model2" @click="search">
              Search
              <v-icon right>mdi-magnify</v-icon>
            </v-btn>
          </v-card-actions>
          <v-expand-transition>
            <v-list v-if="searched" class="blue lighten-3"></v-list>
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
      searched: null
    }
  },
  computed: {
    // fields() {
    //   if (!this.deptSearch) return []

    //   return Object.keys(this.deptSearch).map((key) => {
    //     return {
    //       key,
    //       value: this.deptSearch[key] || 'n/a'
    //     }
    //   })
    // },
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
    deptSearch(val) {
      // Items have already been loaded
      if (this.items.length > 0) return

      // Items have already been requested
      if (this.isLoading) return

      this.isLoading = true

      // Lazily load input items
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
      // this.entries = this.names
      // this.count = this.names.length
      // this.isLoading = false
    },
    termiSearch(val) {
      // Items have already been loaded
      if (this.items.length > 0) return

      // Items have already been requested
      if (this.isLoading) return

      this.isLoading = true

      // Lazily load input items
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
      // this.entries = this.names
      // this.count = this.names.length
      // this.isLoading = false
    }
  },
  methods: {
    search() {
      console.log('search')
      this.searched = true
    }
  }
}
</script>
