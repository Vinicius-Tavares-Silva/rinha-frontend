<template>
  <div class="flex flex-col justify-center">
    <header>
      <h3>JSON Tree Viewer</h3>
    </header>
    <main class="flex flex-col justify-center">
      <span>Simple Json Viewer</span>
      <input type="file" @change="handleFile">
      <span v-if="loadError">invalid File. Please load a valid JSON file</span>
      <section ref="scrollSection" class="content-section">
        <pre>{{ parsedJsonContent }}</pre>
      </section>
    </main>
  </div>
</template>

<script>

const DELIMITER_PAIRS = {
  '{': '}',
  '[': ']',
}
const JSON_OPEN_DELIMITERS = ['{', '[']
const JSON_CLOSE_DELIMITERS = ['}', ']']
const START_CHUNK_SIZE = 84992
const CHUNK_SIZE = 1024

export default {
  name: 'JsonImporter',
  data: () => ({
    file: null,
    fileName: null,
    loadError: false,
    fileContent: '',
    offset: 0,
  }),
  computed: {
    jsonContent: function() {
      return this.buildPartialJson(this.fileContent)
    },
    parsedJsonContent: function() {
      try {
        return JSON.parse(this.jsonContent)
      } catch (error) {
        return this.jsonContent
      }
    }
  },
  mounted() {
    this.$refs.scrollSection.addEventListener("scroll", this.handleScroll);
  },
  destroyed() {
    this.$refs.scrollSection.removeEventListener("scroll", this.handleScroll);
  },
  methods: {
    handleFile(e) {
      this.loadError = false
      this.fileName = null
      const file = e.target.files[0]
      this.file = file
      this.fileName = file.name
      this.readFileAsBinaryString(file, START_CHUNK_SIZE)
    },
    readFileAsBinaryString(file, chunk_size) {
      var fileContent = ''
        const blob = file.slice(this.offset, this.offset + chunk_size);
        const reader = new FileReader();
        reader.onload = e => {
          this.fileContent += e.target.result
          // this.$emit('fileLoaded', { content, fileName: this.fileName})
        }
        reader.readAsBinaryString(blob);
        this.offset += chunk_size;
    },
    handleScroll(e) {
      const scrollSection = this.$refs.scrollSection
      const { scrollTop, clientHeight, scrollHeight } = scrollSection
      if (scrollTop >= (scrollHeight - clientHeight)) {
        this.readFileAsBinaryString(this.file, CHUNK_SIZE)
      }
    },
    buildPartialJson(string) {
      const delimetersOpened = []
      for (const char of string) {
        if (JSON_OPEN_DELIMITERS.includes(char)) {
          delimetersOpened.push(char)
        }
        if (JSON_CLOSE_DELIMITERS.includes(char)) {
          delimetersOpened.pop()
        }
      }
      if (delimetersOpened.length > 0) {
        const delimetersToAdd = delimetersOpened.map(d => DELIMITER_PAIRS[d])
        return string + delimetersToAdd.reverse().join('')
      }
      return string
    }
  }
}
</script>

<style>

.content-section {
  height: 80vh;
  overflow-y: scroll;
}
</style>
