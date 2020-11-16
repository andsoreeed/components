<template>
  <el-menu-item
    class="kb-menu-item"
    :index="`${nodeIndex}`"
    :class="{ active: node && node.path === $route.fullPath }"
    @click="$router.push(node.path)" >
    {{node.title}}
  </el-menu-item>
</template>
<script>
export default {
  props: ['node', 'nodeIndex'],
};
</script>
<style lang="stylus" scope>
.el-menu-item.is-active
  color white !important
</style>

