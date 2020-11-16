<script>
export default {
  render(h) {
    return h(
      'el-form-item',
      {
        class: { 'kb-form-item': true },
        props: {
          ...this.$attrs,
          rules: this.rules,
          prop: this.prop,
        },
        ref: 'elFormItem',
      },
      this.$slots.default,
    );
  },
  props: ['rules', 'prop'],
  inheritAttrs: false,
  methods: {
    resetFiels() {
      this.$refs.elFormItem.resetField();
    },
    clearValidate() {
      this.$refs.elFormItem.clearValidate();
    },
  },
};
</script>
