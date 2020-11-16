<script>
import { isArray, isFunction } from 'lodash';
import globalMixin from '@/mixins/globalMixin';

export default {
  name: 'kbForm',
  mixins: [globalMixin],
  render(h) {
    return h(
      'el-form',
      {
        class: { 'kb-form': true },
        props: {
          model: this.localModel,
          ...this.$attrs,
        },
        on: {
          validate: this.validateHandler,
        },
        ref: 'elForm',
      },
      this.$slots.default,
    );
  },
  inheritAttrs: false,
  props: {
    model: [Array, Object],
    submitMethod: Function,
    initStatus: [Boolean],
  },
  provide() {
    const formData = {};
    Object.defineProperty(formData, 'data', {
      get: () => this.model,
      enumerable: true,
    });
    formData.initStatus = this.localInitStatus;
    return {
      formData,
    };
  },
  data() {
    return {
      localModel: this.methodChangeModel(this.model),
      localInitStatus: {
        value: this.initStatus || false,
        length: 0,
        num: null,
      },
    };
  },
  watch: {
    model: {
      handler() {
        this.localModel = this.methodChangeModel(this.model);
      },
      deep: true,
    },
    initStatus(newVal) {
      this.$nextTick(() => {
        if (this.model.length) {
          this.localInitStatus.length = this.model.length;
          this.localInitStatus.num = this.model.length;
          this.localInitStatus.value = newVal;
        } else this.$emit('update:initStatus', false);
      });
    },
    'localInitStatus.num': {
      handler(newVal) {
        if (newVal === 0) {
          this.$emit('update:initStatus', false);
          this.localInitStatus.num = null;
        }
      },
    },
  },
  methods: {
    methodChangeModel(model) {
      if (isArray(model)) {
        const modelObj = {};
        model.forEach((item, index) => {
          this.$set(modelObj, index, item);
        });
        return modelObj;
      }
      return model;
    },
    validateHandler($event) {
      return this.$emit('validate', $event);
    },
    validate(...params) {
      return this.$refs.elForm.validate(...params);
    },
    validateField(...params) {
      return this.$refs.elForm.validateField(...params);
    },
    resetFields() {
      return this.$refs.elForm.resetFields();
    },
    clearValidate(...params) {
      return this.$refs.elForm.clearValidate(...params);
    },
    submit() {
      this.validate((success) => {
        if (success) {
          if (isFunction(this.submitMethod)) this.submitMethod();
        } else {
          this.fNotify('提示', '请输入必填!', 'error');
        }
      });
    },
  },
};
</script>
