<script setup lang="ts">
import 'bootstrap/scss/bootstrap.scss';

import { ref } from 'vue'
import './Whet.vue'
import Whet from './Whet.vue'
import Process from './Process.vue'
import Footer from './Footer.vue'
</script>

<template>
  <div class="container">
    <div class="row">
      <h1>
        <img src="./whetstone.png" alt="研磨石" class="inline" />
        赤月の森　宝玉シミュレーター
        <img src="./whetstone.png" alt="研磨石" class="inline" />
        　
        <a href="https://twitter.com/intent/tweet?url=https://canvasnote.github.io/akatsuki-simulator/"><img src="./x_post.png" alt="Post to X" /></a>
      </h1>
    </div>
    <div class="row">
      <div class="col container-fluid mb-3">
        <Whet />
      </div>
      <div class="col container-fluid mb-3">
        <Process />
      </div>
    </div>
    <div class="row">
      <Footer />
    </div>
  </div>  
</template>

