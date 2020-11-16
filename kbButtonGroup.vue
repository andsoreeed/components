<script>
export default {
  name: 'kbButtonGroup',
  render(h) {
    return h(
      'el-button-group',
      {
        class: { 'kb-button-group': true },
        props: {
          ...this.$attrs,
        },
      },
      this.$slots.default,
    );
  },
  inheritAttrs: false,
};
</script>
