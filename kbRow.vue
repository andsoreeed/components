<script>
export default {
  render(h) {
    return h(
      'el-row',
      {
        class: { 'kb-row': true },
        props: this.$attrs,
      },
      this.$slots.default,
    );
  },
  inheritAttrs: false,
};
</script>

