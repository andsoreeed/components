<script>
export default {
  render(h) {
    return h(
      'el-card',
      {
        class: { 'kb-card': true },
        props: this.$attrs,
      },
      this.$slots.default,
    );
  },
  inheritAttrs: false,
};
</script>
