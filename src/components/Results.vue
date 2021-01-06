<template>
  <v-container>
    <transition-group name="list" class="row justify-center">
      <div
        v-for="(gif,index) in gifs"
        :key="`gif-${gif.title}-${index}`"
        class="col-12 col-sm-6 col-md-3 text-center mt-4"
      >
        <h2 v-if="notfound" >Not Found</h2>
        <v-img
          :lazy-src="loader"
          :src="gif.url"
        >
        <v-btn
          @click.prevent="WebShareInit(gif)"
          x-small
          depressed
          dark
          fab
          color="rgba(0,0,0,0.5)"
          class="float-right mr-2 mt-2"
        >
          <v-icon dark>
            mdi-reply
          </v-icon>
        </v-btn>
        </v-img>
      </div>
    </transition-group>
  </v-container>
</template>

<script lang="ts">

  import Vue from 'vue'
  import {GifData} from '@/common/types'

  export default Vue.extend({
    name: 'Results',

    data: () => ({
      loader: require('@/assets/loading1.gif')  
    }),

    props: {
      gifs: {
        type: Array,
        default: () => ([])
      },
      notfound: {
        type: Boolean,
        default: false
      }
    },

    methods: {
      WebShareInit(gif: GifData) {
        if (navigator.share) {
          navigator.share({
            title: gif.title,
            text: gif.title,
            url: gif.url,
          })
          .then(function () {
              console.log("Shareing successfull")
          })
          .catch(function () {
              console.log("Sharing failed")
          })
        }
        else {
          alert('share not supported in your browser')
        }
      }
    }
  })
</script>

<style lang="scss">

  .list-enter-active, .list-leave-active {
    transition: all 1s;
  }

  .list-enter, .list-leave-to {
    opacity: 0;
    transform: translateX(20px);
  }

</style>