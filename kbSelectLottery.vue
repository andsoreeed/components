<template>
  <kb-col class="kb-select kb-select-lottery">
    <template v-if="info.formType">
      <kb-form-item
        :label="info.label"
        :prop="info.validateKey"
        :rules="info.validateRules"
        ref="formItem" >
        <kb-select-lottery-inner
          v-model="value"
          :info="info"
          v-bind="$attrs" />
      </kb-form-item>
    </template>
    <template v-else>
      <label v-if="info.label">{{info.label}}</label>
      <kb-select-lottery-inner
        v-model="value"
        :info="info"
        v-bind="$attrs" />
    </template>
  </kb-col>
</template>
<script>
import { isArray } from 'lodash';
import kbSelectLotteryInner from './kbSelectLotteryInner.vue';
import kbCol from './kbCol.vue';
import kbFormItem from './kbFormItem.vue';

export default {
  name: 'kbSelectLottery',
  inheritAttrs: false,
  components: {
    kbSelectLotteryInner,
    kbCol,
    kbFormItem,
  },
  model: {
    prop: 'propValue',
    event: 'valueChange',
  },
  props: {
    propValue: {
      required: true,
    },
    info: {
      required: true,
      default: () => {},
    },
  },
  data() {
    return {
      value: this.propValue,
    };
  },
  watch: {
    value(val) { this.$emit('valueChange', val); },
    propValue: {
      handler(val) {
        if (isArray(val) && isArray(this.value) && val.join('') === this.value.join('')) return;
        else if (!isArray(val) && !isArray(this.value) && val === this.value) return;
        this.value = val;
      },
      immediate: true,
    },
  },
};
</script>
