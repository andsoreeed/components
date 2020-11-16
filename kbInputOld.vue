<template>
  <kb-col
    class="kb-input"
    :class="{ [`${info.className}`]: !!info.className }" >
    <template v-if="info.formType">
      <!-- TODO -->
      <!-- 驗證表單 -->
      <kb-form-item
        :label="info.label"
        :prop="info.validateKey"
        :rules="info.validateRules"
        ref="formItem" >
        <el-input
          v-model="value"
          :placeholder="info.placeholder || ''"
          :autosize="$attrs.type === 'textarea' ? true : false"
          v-bind="$attrs"
          :disabled="has($attrs, 'disabled') ? $attrs.disabled : info.disabled" />
        <slot/>
      </kb-form-item>
    </template>
    <template v-else>
      <label v-if="info.label">
        {{info.label}}
      </label>
      <div class="el-input-box">
        <el-input
          v-model="value"
          :placeholder="info.placeholder || ''"
          :autosize="$attrs.type === 'textarea' ? true : false"
          v-bind="$attrs"
          :disabled="has($attrs, 'disabled') ? $attrs.disabled : info.disabled"
          @input="methodHandleInput(value, info.method)" >
          <template
            v-if="info.prependLabel"
            slot="prepend">
            {{info.prependLabel}}
          </template>
          <template
            v-if="info.appendLabel"
            slot="append">
            {{info.appendLabel}}
          </template>
        </el-input>
        <slot/>
      </div>
      <div><slot name="note"></slot></div>
    </template>
  </kb-col>
</template>
<script>
import { isArray, has } from 'lodash';
import kbFormItem from './kbFormItem.vue';
import kbCol from './kbCol.vue';

export default {
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
      // validateStatus: '',
    };
  },
  watch: {
    // validateStatus(val) {
    //   console.log('trigger type', val);
    //   if (val) {
    //     this.$nextTick(() => {
    //       this.$refs.formItem.$refs.elFormItem.validate(val);
    //       this.validateStatus = '';
    //     });
    //   }
    // },
    value(val) {
      if (val !== this.propValue) {
        if (this.$attrs.type === 'number') {
          this.$emit('valueChange', Number(val));
        } else {
          this.$emit('valueChange', val);
        }
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
    // TODO:
    // 由handleInput接手input事件處理
    methodHandleInput(value, method) {
      if (this.$attrs.type === 'number') {
        this.$emit('input', Number(value));
      } else {
        this.$emit('input', value);
      }
      if (method) {
        method();
      }
    },
    // TODO: blur and focus
    // checkTrigger(type) {
    // const tmp = [].concat(this.info.validateRules);
    // for (let i = 0, len = tmp.length; i < len; i+= 1) {
    //   const str = typeof tmp[i].trigger !== 'string' ? tmp[i].trigger.join('') : tmp[i].trigger;
    //   if (str.includes(type)) break;
    // }
    // tmp.forEach((item) => {
    //   const str = typeof item.trigger !== 'string' ? item.trigger.join('') : item.trigger;
    //   if (str.includes(type)) {
    //     this.validateStatus = type;
    //     return true;
    //   }
    //   return false;
    // });
    // },
  },
};
</script>
