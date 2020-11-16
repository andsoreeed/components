<template>
  <el-submenu :index="nodeIndex">
    <template slot="title"><span>{{node.title}}</span></template>
    <slot />
  </el-submenu>
</template>
<script>
export default {
  props: ['node', 'nodeIndex'],
};
</script>

