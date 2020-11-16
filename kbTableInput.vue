<script>
import { isFunction, cloneDeep } from 'lodash';
import kbInput from './kbInput.vue';

export default {
  props: ['item', 'scope'],
  data() {
    return {
      value: this.formData.data[this.scope.$index][this.item.dataKey] || '',
    };
  },
  computed: {
    localInputSetting() {
      const tmp = cloneDeep(this.item.inputSetting);
      if (tmp && tmp.formType && tmp.validateKey === '$index') {
        tmp.validateKey = this.scope.$index.toString();
        return tmp;
      }
      return tmp;
    },
  },
  components: {
    kbInput,
  },
  inject: {
    formData: {
      default: () => null,
    },
  },
  methods: {
    handleInput($event, method) {
      if (method && isFunction(method)) method(this.scope.$index, $event);
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
    return h('kb-input', {
      props: {
        info: this.localInputSetting,
        propValue: this.value,
      },
      attrs: {
        type: this.item.inputType,
      },
      on: {
        input: ($event) => {
          this.handleInput($event, this.item.tableMethod);
        },
        valueChange: ($event) => {
          this.value = $event;
        },
      },
      ref: 'input',
    });
  },
};
</script>
