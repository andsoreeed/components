<script>
import { has, isFunction, isRegExp } from 'lodash';
import kbFormItem from './kbFormItem.vue';
import kbCol from './kbCol.vue';

export default {
  render(h) {
    function hElInput(hInput, self) {
      return hInput(
        'el-input', {
          on: {
            input($event) {
              self.methodHandleInput($event, self.info.method);
              self.$emit('valueChange', self.methodChangeEmitType($event));
            },
          },
          nativeOn: {
            keypress($event) {
              // 功能鍵白名單
              // const whiteListKeyCode = [8, 9, 13, 16, 17, 18, 37, 38, 39, 40, 46, 91, 93];
              // FIXME: 暫時先把 . 加入白名單
              const whiteListKeyCode = [46];
              if (whiteListKeyCode.includes($event.keyCode)) return true;
              const regStr = $event.key;
              if (self.info.regRule &&
                isRegExp(self.info.regRule) &&
                !self.info.regRule.test(regStr)) {
                $event.preventDefault();
                return false;
              }
              return true;
            },
          },
          props: {
            autosize: self.$attrs.type === 'textarea',
            disabled: has(self.attrs, 'disabled') ? self.$attrs.disabled : self.info.disabled,
            ...self.$attrs,
            value: self.value,
          },
          attrs: {
            placeholder: (self.info.placeholder) || '',
            ...self.info.attrs,
            size: 'small',
          },
        },
        ['prepend', 'append'].map((item) => {
          if (self.info[`${item}Label`]) {
            return hInput(
              'template',
              {
                slot: item,
              },
              self.info[`${item}Label`],
            );
          }
          return hInput();
        }),
      );
    }
    function hElInputWrapper(hWrapper, self) {
      if (self.info.formType) {
        return hWrapper(
          'kb-form-item', {
            props: {
              prop: self.info.validateKey,
              rules: self.info.validateRules,
            },
            attrs: {
              label: self.info.label,
            },
            ref: 'formItem',
          },
          [hElInput(hWrapper, self), self.$slots.default],
        );
      }
      const tmp = [];
      if (self.info.label) tmp.push(hWrapper('label', [self.info.label]));
      return [
        ...tmp,
        hWrapper(
          'div', {
            class: {
              'el-input-box': true,
            },
          },
          [hElInput(hWrapper, self), self.$slots.default],
        ),
        hWrapper(
          'div', {
            slot: 'note',
          },
          self.$slots.default,
        ),
      ];
    }
    return h(
      'kb-col', {
        class: {
          'kb-input': true,
          [`${this.info.className}`]: !!this.info.className,
        },
      },
      [hElInputWrapper(h, this)],
    );
  },
  inheritAttrs: false,
  model: {
    prop: 'propValue',
    event: 'valueChange',
  },
  props: {
    propValue: {
      required: true,
    },
    info: {
      type: Object,
      default: () => ({}),
    },
  },
  components: {
    kbFormItem,
    kbCol,
  },
  data() {
    return {
      value: this.propValue,
    };
  },
  watch: {
    value(val) {
      if (val !== this.propValue) {
        this.$emit('valueChange', this.methodChangeEmitType(val));
      }
    },
    propValue(val) {
      if (val !== this.value) {
        this.value = this.propValue;
      }
    },
  },
  methods: {
    // lodash
    has,
    methodChangeEmitType(val) {
      if ((this.$attrs.type === 'number')
        || (this.info.attrs && this.info.attrs.type === 'number')) return Number(val);
      return val;
    },
    methodHandleInput(val, method) {
      this.$emit('input', this.methodChangeEmitType(val));
      if (method && isFunction(method)) method();
    },
  },
};
</script>
