<template>
  <q-page class="flex flex-center">
    <Canvas/>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue';
import Canvas from '../components/Canvas.vue'

export default defineComponent({
  components: {
    Canvas,
  },
  name: 'PageIndex'
})
</script>

<style scoped>
body {
  margin: auto;
}
</style>
