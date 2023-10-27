<template>
  <li class="node-tree">
    <ul v-if="isArray(node)" class="node-list">
      <li v-for="(item, index) in node" :key="index">
        <span class="node-value">{{ index }}: </span>
        <span v-if="notInterrable(item)" class="node-value">{{ item }}</span>
        <node-tree v-else :node="item" />
      </li>
    </ul>
    <ul v-else-if="isObject(node)">
      <li v-for="(value, key) in node" :key="key">
        <span class="node-key">"{{ key }}": </span>
        <span v-if="notInterrable(value)" class="node-value">{{ value }}</span>
        <node-tree v-else :node="value" />
      </li>
    </ul>
  </li>
</template>

<script>
export default {
  name: "node",
  props: ['node'],
  methods: {
    isArray: function(item) {
      return Array.isArray(item)
    },
    isObject: function(item) {
      return typeof item === "object"
    },
    notInterrable: function(item) {
      return !this.isArray(item) && !this.isObject(item)
    }
  },
};
</script>

<style>
.node-key {
  color: cadetblue;
}
</style>