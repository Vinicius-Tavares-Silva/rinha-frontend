<template>
  <div class="flex flex-col justify-center">
    <div v-if="!fileLoaded" class="mt-80">
      <header class="text-center">
        <h1 class="font-bold text-6xl m-2">JSON Tree Viewer</h1>
      </header>
      <main class="flex flex-col justify-center text-center">
        <span class="text-2xl m-2">Simple Json Viewer that runs completely on-client. No data exchange</span>
        <div class="flex justify-center m-2">
          <label 
            for="fileInput" 
            class="border-2 border-slate-800 bg-slate-200 p-2 rounded shadow-md hover:bg-slate-300 font-medium cursor-pointer"
          >
            Load JSON
          </label>
          <input name="fileInput" id="fileInput" accept="application/JSON" type="file" @change="handleFile">
        </div>
        <span v-if="loadError" class="text-red-600 text-lg">invalid File. Please load a valid JSON file</span>
      </main>
    </div>
    <div v-else>
      <header class="flex justify-center text-center">
        <button type="reset" class="mt-4 mb-2 hover:scale-110" @click="resetData">
          <span class="text-4xl">&#8592;</span>
        </button>
        <h1 class="font-bold text-4xl">{{ fileName }}</h1>
      </header>
    </div>
    <section ref="scrollSection" class="ml-8" :class="fileLoaded && 'content-section'">
      <Tree v-if="parsedJsonContent" :treeData="parsedJsonContent" />
    </section>
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
const CHUNK_SIZE = 2048

export default {
  name: 'JsonImporter',
  data: () => ({
    file: null,
    fileName: null,
    fileContent: '',
    fileSize: null,
    loadError: false,
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
      if (this.file && this.jsonContent) {
        try {
          return JSON.parse(this.jsonContent)
        } catch (error) {
          console.log(error);
          // console.log(this.jsonContent);
          return 'error'
        }
      }
      return null
    },
    fileLoaded: function() {
      return this.file && this.parsedJsonContent
    }
  },
  watch: {
    parsedJsonContent(newValue) {
      if (newValue === 'error') {
        // this.$emit('fileLoaded', { content: newValue, fileName: this.fileName})
        this.loadError = true
      }
    },
    offset(newValue) {
      if (newValue >= this.fileSize) {
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
      this.fileSize = file.size
      this.readFileAsBinaryString(file, CHUNK_SIZE, file.size)
    },
    readFileAsBinaryString(file, chunk_size, fileSize) {
      var fileContent = ''
      const newOffset = this.offset + chunk_size
      const blob = file.slice(this.offset, newOffset);
      const reader = new FileReader();
      reader.onload = e => {
        if (!this.fileContent) {
          if (newOffset >= fileSize) {
            try {
              JSON.parse(e.target.result)
            } catch (error) {
              this.loadError = true
              return
            }
          }
        }
        this.fileContent += e.target.result
        // this.$emit('fileLoaded', { content, fileName: this.fileName})
      }
      if (this.offset <= this.fileSize) {
        reader.readAsBinaryString(blob);
        this.offset += chunk_size;
      }
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
      this.fileName = null,
      this.fileSize = null,
      this.fileContent = ''
      this.offset = 0,
      this.loadError = false
    },
  }
}
</script>

<style>
.content-section {
  height: 80vh;
  overflow-y: auto;
}
input[type="file"] {
    display: none;
}
</style>
