<script setup lang="ts">
import { ref } from 'vue'
import './Whet.vue'
import Whet from './Whet.vue'
import Process from './Process.vue'
</script>

<template>
  <h1>
    <img src="./whetstone.png" alt="研磨石" class="inline" />
    赤月の森　宝玉シミュレーター
    <img src="./whetstone.png" alt="研磨石" class="inline" />
  </h1>
  <Whet />
  <Process />
  
</template>

