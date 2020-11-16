<script>
import kbSwitch from './kbSwitch.vue';

export default {
  props: ['item', 'scope'],
  data() {
    return {
      value: this.formData.data[this.scope.$index][this.item.dataKey] || '',
    };
  },
  components: {
    kbSwitch,
  },
  inject: {
    formData: {
      default: () => null,
    },
  },
  watch: {
    value(newVal) {
      this.formData.data[this.scope.$index][this.item.dataKey] = newVal;
    },
    'formData.initStatus.value': {
      handler(newVal) {
        if (newVal) {
          this.value = this.formData.data[this.scope.$index][this.item.dataKey] || '';
          this.formData.initStatus.num -= 1;
        }
      },
    },
  },
  render(h) {
    return h('kb-switch', {
      props: {
        info: this.item.switchSetting,
        propValue: this.value,
      },
      on: {
        valueChange: ($event) => {
          this.value = $event;
        },
        change: () => {
          if (this.item.handleChange) this.item.handleChange(this.item, this.scope);
        },
      },
    });
  },
};
</script>
