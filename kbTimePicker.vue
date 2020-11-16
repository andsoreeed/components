<template>
  <kb-col class="kb-time-picker">
    <label v-if="info.label">
      {{info.label}}
    </label>
    <div class="row">
      <el-time-picker
        v-model="value"
        :placeholder="info.placeholder || ''"
        :value-format="info.valueFormat"
        v-bind="$attrs"
        size="small" />
      <div v-if="info.note !== ''">{{info.note}}</div>
    </div>
  </kb-col>
</template>
<script>
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
      required: true,
    },
  },
  components: {
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
        this.$emit('valueChange', val);
      }
    },
    propValue(val) {
      if (val !== this.value) {
        this.value = this.propValue;
      }
    },
  },
};
</script>
