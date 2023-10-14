<template>
  <div class="inline-flex flex-col">
    <button @click="handleClick">Return</button>
    <span>{{ fileName }}</span>
    <div>
      <pre>{{ fileContent }}</pre>
    </div>
  </div>
</template>

<script>
export default {
  name: 'JsonViewer',
  props: {
    file: {
      type: Object,
      required: true
    }
  },
  computed: {
    fileContent() {
      return this.file.content
    },
    fileName() {
      return this.file.fileName
    }
  },
  methods: {
    handleClick() {
      this.$emit('return')
    }
  }
}
</script>
