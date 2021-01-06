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
          @input="pauseEnter"
          autofocus
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
import {GifData} from '@/common/types'

export default Vue.extend({
  name: 'App',
  data: () => ({
    data: [] as Array<GifData>,
    notFound: false as boolean,
    query: '' as string,
    limit: 30 as number,
    loader: require('@/assets/loading1.gif'),
    loadTimeout: false as boolean,
    blockEnter: false as boolean,
  }),

  components: {
    Results: () => import('./components/Results.vue'),
  },

  created() {
    this.GetRandomGifs(this.limit).then((data: Array<GifData>) => this.data = data) 
  },

  methods: {

    pauseEnter($event: string) {
      $event = $event.trim()
      this.query = $event

      setTimeout(() => {
        if($event === this.query) {
          this.pushReq(this.query)
        }
      }, 600)
    },

    pushReq(query: string) {
      
      if(query === '') {
        this.GetRandomGifs(30).then( (data: Array<GifData>) => this.data = data)
        this.notFound = false
        return
      }

      fetch(`http://api.giphy.com/v1/gifs/search?q=${query}&api_key=aeXjbr8K3v1V83Pw0yJySLOhXMGeh33B&limit=30`)
      .then(res => {
        if(!res.ok) {
          throw Error(res.statusText)
        }
        return res.json()
      })
      .then(data => {
        console.log(data)
        if(data.data.length) {
          this.data = data.data.map( (item: any) => ({url: item.images.original.url,title: item.title}))
          this.notFound = false
        }
        else {
          this.GetRandomGifs(1).then( (data: Array<GifData>) => {
            this.notFound = true
            this.data = data; 
          })
        }

        this.limit = 30
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
        return data.data.map( <GifData extends {}>(item: any) => ({url: item.images.original.url, title: item.title}))
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
              this.data.push({url: item.images.original.url,title: item.title})
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
              this.data.push({url: item.images.original.url,title: item.title})
            });
          })
        }

      }
    }
  }
});
</script>