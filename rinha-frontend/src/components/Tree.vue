<template>
  <div class="tree">
    <ul class="tree-list">
      <node-tree :node="treeData"></node-tree>
    </ul>
  </div>
</template>

<script>
import NodeTree from "./NodeTree.vue";

export default {
  props: {
    treeData: Object
  },
  components: {
    NodeTree
  },
};
</script>

<style>
.tree-list ul {
  padding-left: 24px;
  margin: 6px 0;
}
</style>