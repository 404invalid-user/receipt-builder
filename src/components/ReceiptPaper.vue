<script setup>
import { ref } from 'vue'

defineProps({
  receipt: Object
})

const count = ref(0)
</script>

<template>
  <div class="paper">
    
  </div>
</template>

<style scoped>
.paper {
  background-color: #FFFFFF;
  border: 0.9px solid black;
  width: 410px;
  margin: auto 0;

  min-height: 100px;
}
</style>
