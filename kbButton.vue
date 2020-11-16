<script>
import { mapState } from 'vuex';

export default {
  render(h) {
    if (!this.buttonName || (this.permissionTree && this.permissionTree.has(this.buttonName))) {
      return h(
        'el-button',
        {
          class: { 'kb-button': true },
          props: this.$attrs,
          on: {
            click: this.clickHandler,
          },
        },
        this.$slots.default,
      );
    } else if (this.buttonBackup) {
      return h(
        'div',
        {},
        this.$slots.default,
      );
    }
    return h;
  },
  inheritAttrs: false,
  props: {
    buttonName: {
      type: String,
      default: '',
    },
    buttonBackup: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    ...mapState({
      permissionTree: state => state.global.permissionTree,
    }),
  },
  methods: {
    clickHandler() {
      this.$emit('click');
    },
  },
};
</script>
