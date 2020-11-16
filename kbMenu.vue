<template>
  <el-menu
    class="kb-menu"
    :mode="mode"
    :default-active="defaultActivePath"
    :default-openeds="defaultOpeneds"
    :unique-opened="true"
    :collapse="false"
    background-color="rgba(0,0,0,.1)"
    text-color="white"
    active-text-color="#44a9c5" >
    <div class="menu-wrapper">
      <slot />
    </div>
  </el-menu>
</template>
<script>
export default {
  props: {
    mode: {
      type: String,
    },
    defaultActivePath: {
      type: String,
    },
    defaultOpeneds: {
      type: Array,
      default: () => [],
    },
  },
};
</script>

