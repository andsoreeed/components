<script>
import kbSelect from './kbSelect.vue';

export default {
  props: ['item', 'scope'],
  data() {
    return {
      value: this.formData.data[this.scope.$index][this.item.dataKey]
        || (this.item.multiple ? [] : ''),
    };
  },
  components: {
    kbSelect,
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
          this.value = this.formData.data[this.scope.$index][this.item.dataKey]
            || (this.item.multiple ? [] : '');
          this.formData.initStatus.num -= 1;
        }
      },
    },
  },
  render(h) {
    return h('kb-select', {
      props: {
        info: this.item.selectSetting,
        propValue: this.value,
      },
      on: {
        valueChange: ($event) => {
          this.value = $event;
        },
      },
    });
  },
};
</script>
