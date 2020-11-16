<script>
export default {
  render(h) {
    return h(
      'el-col',
      {
        class: { 'kb-col': true },
        props: this.$attrs,
      },
      this.$slots.default,
    );
  },
  inheritAttrs: false,
};
</script>
