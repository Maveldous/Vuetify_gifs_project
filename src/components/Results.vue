<template>
  <v-container>
    <transition-group name="list" class="row justify-center">
      <div
        v-for="(gif,index) in gifs"
        :key="`gif-${gif}-${index}`"
        class="col-12 col-sm-6 col-md-3 text-center mt-4"
      >
        <h2 v-if="notfound" >Not Found</h2>
        <v-img
          :lazy-src="loader"
          :src="gif"
        />
      </div>
    </transition-group>
  </v-container>
</template>

<script lang="ts">
  import Vue from 'vue'

  export default Vue.extend({
    name: 'results',

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
  })
</script>

<style lang="scss">
  .list-enter-active, .list-leave-active {
    transition: all 1s;
  }
  .list-enter, .list-leave-to {
    opacity: 0;
  }
  .list-leave-active {
    position: absolute;
  }
</style>