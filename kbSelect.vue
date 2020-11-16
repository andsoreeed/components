<template>
  <kb-col
    class="kb-select"
    :class="{ [`${info.className}`]: !!info.className }" >
    <template  v-if="info.formType">
      <kb-form-item
        :label="info.label"
        :prop="info.validateKey"
        :rules="info.validateRules"
        ref="formItem">
        <el-select
          v-model="value"
          :placeholder="info.placeholder || ''"
          :filterable="info.filterable"
          :multiple="info.multiple"
          v-bind="$attrs"
          @change="methodSelectValueChange(value, info.method)"
          @input="handleInput"
          size="small"
          :popper-class="selectListMultiLine" >
          <el-option
            v-for="item in info.options"
            :key="item.id"
            :label="item.value"
            :value="item.id"
            ref="elSelect" />
        </el-select>
      </kb-form-item>
    </template>
    <template v-else>
      <label v-if="info.label">
        {{info.label}}
      </label>
      <el-select
        v-model="value"
        :placeholder="info.placeholder || ''"
        :filterable="info.filterable"
        :multiple="info.multiple"
        v-bind="$attrs"
        @change="methodSelectValueChange(value, info.method)"
        ref="elSelect"
        size="small"
        :popper-class="selectListMultiLine" >
        <el-option
          v-for="item in info.options"
          :key="item.id"
          :label="item.value"
          :value="item.id" />
      </el-select>
    </template>
  </kb-col>
</template>
<script>
import { isFunction, isArray } from 'lodash';
import kbFormItem from './kbFormItem.vue';
import kbCol from './kbCol.vue';

export default {
  inheritAttrs: false,
  model: {
    prop: 'propValue',
    event: 'valueChange',
  },
  props: {
    // v-model綁定值
    propValue: {
      required: true,
      default: '',
    },
    info: {
      type: Object,
      required: true,
    },
  },
  attrs: {
    size: 'small',
  },
  components: {
    kbFormItem,
    kbCol,
  },
  data() {
    return {
      value: this.propValue,
      valueLabel: '',
    };
  },
  computed: {
    option() {
      if (isArray(this.info.options)) return this.info.options.find(opt => opt.id === this.value);
      return null;
    },
    selectListMultiLine() {
      return this.info.className === 'multi-line' ? 'kb-select-dropdown__multi-line' : '';
    },
  },
  mounted() {
    if (this.value) {
      this.methodSelectValueChange(this.value, this.info.method);
      this.valueLabel = this.$refs.elSelect ? this.$refs.elSelect.selectedLabel : '';
    }
    if (isFunction(this.info.options)) {
      this.getOptions();
    }
  },
  watch: {
    value(val) {
      this.$emit('valueChange', val);
      this.$nextTick(() => {
        this.valueLabel = this.$refs.elSelect ? this.$refs.elSelect.selectedLabel : '';
      });
    },
    propValue: {
      handler(val) {
        if (!isArray(val) && val && isArray(this.info.options) &&
          !this.info.options.find(option => option.id === val)) {
          this.$emit('valueChange', null);
        } else if (val !== this.value) {
          this.value = this.propValue;
          this.$nextTick(() => {
            this.valueLabel = this.$refs.elSelect ? this.$refs.elSelect.selectedLabel : '';
          });
          this.methodSelectValueChange(this.value, this.info.method);
        }
      },
      immediate: true,
    },
    info: {
      async handler(val) {
        const res = await (isFunction(this.info.options) ? this.getOptions() : this.info.options);
        if (
          this.info.checkOptions &&
          !this.info.options.some(item => item.id === this.value)
        ) {
          if (this.info.multiple) this.value.splice(0);
          else this.value = null;
        }
        this.setMultiDropdownListWidth();
      },
      deep: true,
    },
  },
  methods: {
    async methodSelectValueChange(value, method) {
      if (method && value !== null && value !== undefined) {
        const res = await method(value);
        if (res) {
          if (!res.code) {
            this.$nextTick(() => { });
          }
        }
      }
    },
    async getOptions() {
      if (this.value) {
        const tmp = this.value;
        this.value = isArray(tmp) ? [] : null;
        this.info.options = await this.info.options();
        if (this.value === [] || this.value === null) this.value = tmp;
        else if (!isArray(this.value) &&
          !this.info.options.find(option => option.id === this.value)) {
          this.$emit('valueChange', null);
        }
        return;
      }
      this.info.options = await this.info.options();
    },
    // TODO:
    handleInput(value) {
      if (Array.isArray(this.info.validateRules)) {
        if (Array.isArray(this.info.validateRules[0].trigger)) {
          if (this.info.validateRules[0].trigger.includes('input')) {
            if (value !== this.propValue) {
              this.$emit('valueChange', value);
              this.$refs.formItem.$refs.elFormItem.validate('input');
            }
          }
        } else if (this.info.validateRules[0].trigger === 'input') {
          if (value !== this.propValue) {
            this.$emit('valueChange', value);
            this.$refs.formItem.$refs.elFormItem.validate('input');
          }
        }
      } else {
        console.log('%c 验证格式错误 ', 'background: red; color: white; font-size:20px;');
      }
    },
    setMultiDropdownListWidth() {
      document.querySelectorAll('.kb-select-dropdown__multi-line').forEach((item) => {
        const itemMinWidth = item.style.minWidth;
        if (itemMinWidth) {
          item.style.maxWidth = itemMinWidth;
        }
      });
    },
  },
};
</script>
