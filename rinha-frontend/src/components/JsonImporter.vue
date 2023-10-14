<template>
  <div class="flex flex-col justify-center">
    <header>
      <h3>JSON Tree Viewer</h3>
    </header>
    <main class="flex flex-col justify-center">
      <span>Simple Json Viewer</span>
      <input type="file" @change="handleFile">
      <span v-if="loadError">invalid File. Please load a valid JSON file</span>
    </main>
  </div>
</template>

<script>
export default {
  name: 'JsonImporter',
  data: () => ({
    fileName: null,
    loadError: false,
  }),
  methods: {
    handleFile(e) {
      this.loadError = false
      this.fileName = null
      const file = e.target.files[0]
      this.fileName = file.name
      this.readFile(file)
    },
    readFile(file) {
      const reader = new FileReader()
      reader.onload = e => {
        try {
          const content = JSON.parse(e.target.result)
          this.$emit('fileLoaded', { content, fileName: this.fileName})
          
        } catch (error) {
          this.loadError = true
        }
      }
      reader.readAsText(file)
    }
  }
}
</script>

<style>
/* Your component's CSS goes here */
</style>
