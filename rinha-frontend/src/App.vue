
<template>
  <JsonViewer v-if="file" @return="handleReturn" :file="file" />
  <JsonImporter v-else @fileLoaded="handleFileLoaded" />
</template>

<script>
import JsonViewer from './components/JsonViewer.vue'
import JsonImporter from './components/JsonImporter.vue'

export default {
  name: 'App',
  components: {
    JsonViewer,
    JsonImporter
  },
  data: () => ({
    file: null
  }),
  methods: {
    handleFileLoaded(file) {
      this.file = file
    },
    handleReturn() {
      this.file = null
    }
  }
}
</script>

<style scoped>

</style>
