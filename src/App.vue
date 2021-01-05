<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
    <v-row>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>
      
      <v-spacer></v-spacer>

      <v-col
        cols="6"
        md="3"
      >

        <v-text-field
          @input="pushReq"
          append-icon="mdi-magnify"
          light
          type="text"
          :hide-details="true"
          :rounded="true"
          background-color="white"
        />

      </v-col>


    </v-row>


    </v-app-bar>

    <v-main style="padding-top: 150px;">
      <Results v-scroll="infinityScrollInit" :gifs="data" :notfound="notFound"/>
    </v-main>
  </v-app>
</template>

<script lang="ts">

import Vue from 'vue';

export default Vue.extend({
  name: 'App',
    data: () => ({
      data: [] as any,
      notFound: false as boolean,
      query: '' as string,
      limit: 20 as number,
      loader: require('@/assets/loading1.gif'),
      loadTimeout: false as boolean,
    }),

  components: {
    Results: () => import('./components/Results.vue'),
  },

  created() {
    this.GetRandomGifs(this.limit).then((data: any) => this.data = data) 
  },

  methods: {

    pushReq($event: string) {
      this.query = $event

      if($event === '') {
        this.GetRandomGifs(20).then( (data: any) => this.data = data)
        this.notFound = false
        return
      }

      fetch(`http://api.giphy.com/v1/gifs/search?q=${$event}&api_key=aeXjbr8K3v1V83Pw0yJySLOhXMGeh33B&limit=20`)
      .then(res => {
        if(!res.ok) {
          throw Error(res.statusText)
        }
        return res.json()
      })
      .then(data => {
        console.log(data)
        if(data.data.length) {
          this.data = data.data.map( (item: any) => item.images.original.url)
          this.notFound = false
        }
        else {
          this.GetRandomGifs(1).then( (data: any) => this.data = data)
          this.notFound = true
        }

        this.limit = 20
      })
    },

    GetRandomGifs(limit: number): any {
      return fetch(`http://api.giphy.com/v1/gifs/trending?api_key=aeXjbr8K3v1V83Pw0yJySLOhXMGeh33B&limit=${limit}`)
      .then(res => {
        if(!res.ok) {
          throw Error(res.statusText)
        }
        return res.json()
      }).then(data => {
        return data.data.map( (item: any) => item.images.original.url)
      })
    },

    infinityScrollInit(evt: any, el: any) {
      if (document.documentElement.getBoundingClientRect().bottom - document.documentElement.clientHeight < 100 && !this.loadTimeout) {

        this.loadTimeout = true
        setTimeout( () => this.loadTimeout = false, 3000)
        this.limit += 40
        
        if(this.query) {
          fetch(`http://api.giphy.com/v1/gifs/search?q=${this.query}&api_key=aeXjbr8K3v1V83Pw0yJySLOhXMGeh33B&limit=40&offset=${this.limit}`)
          .then(res => {
            if(!res.ok) {
              throw Error(res.statusText)
            }
            return res.json()
          })
          .then(data => {
            data.data.forEach((item: any) => {
              this.data.push(item.images.original.url)
            });
          })
        }
        else {
          return fetch(`http://api.giphy.com/v1/gifs/trending?api_key=aeXjbr8K3v1V83Pw0yJySLOhXMGeh33B&limit=40&offset=${this.limit}`).then(res => {
            if(!res.ok) {
              throw Error(res.statusText)
            }
            return res.json()
          })
          .then(data => {
            data.data.forEach((item: any) => {
              this.data.push(item.images.original.url)
            });
          })
        }

      }
    }
  }
});
</script>