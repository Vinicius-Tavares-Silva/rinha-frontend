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
        <Tree v-if="parsedJsonContent" :treeData="parsedJsonContent" />
      </section>
    </main>
  </div>
</template>

<script>
import Tree from './Tree.vue'

const DELIMITER_PAIRS = {
  '{': '}',
  '[': ']',
}
const JSON_OPEN_DELIMITERS = ['{', '[']
const JSON_CLOSE_DELIMITERS = ['}', ']']
const START_CHUNK_SIZE = 2048
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
  components: {
    Tree,
  },
  computed: {
    jsonContent: function() {
      return this.buildPartialJson(this.fileContent)
    },
    parsedJsonContent: function() {
      if (this.file) {
        try {
          return JSON.parse(this.jsonContent)
        } catch (error) {
          console.log(error);
          // console.log(this.jsonContent);
          return null
        }
      }
    }
  },
  watch: {
    parsedJsonContent: function(newValue) {
      if (newValue) {
        // this.$emit('fileLoaded', { content: newValue, fileName: this.fileName})
      } else {
        this.loadError = true
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
      this.resetData()
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
      let i = 0
      let lastDelimeterOpenedIndex = 0
      let lastDelimeterClosedIndex = 0
      let insideString = false
      const delimetersOpened = []
      for (const char of string) {
        if (char === '"') {
          insideString = !insideString
        }
        if (JSON_OPEN_DELIMITERS.includes(char) && !insideString) {
          lastDelimeterOpenedIndex = i
          delimetersOpened.push(char)
        }
        if (JSON_CLOSE_DELIMITERS.includes(char) && !insideString) {
          lastDelimeterClosedIndex = i
          delimetersOpened.pop()
        }
        i += 1
      }
      if (delimetersOpened.length > 0) {
        const sliceIndex = lastDelimeterClosedIndex > lastDelimeterOpenedIndex ? lastDelimeterClosedIndex : lastDelimeterOpenedIndex
        const delimetersToAdd = delimetersOpened.map(d => DELIMITER_PAIRS[d])
        const formattedString = sliceIndex === (i-1) ? string : string.slice(0, sliceIndex - i + 1)
        return formattedString + delimetersToAdd.reverse().join('')
      }
      return string
    },
    resetData() {
      this.file = null
      this.fileName = null
      this.fileContent = ''
      this.offset = 0
    },
  }
}
</script>

<style>

.content-section {
  height: 80vh;
  overflow-y: scroll;
}
</style>
