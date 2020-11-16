<template>
  <kb-col class="kb-switch">
    <label v-if="info.label">{{info.label}}</label>
    <el-switch
      v-model="value"
      :active-value="info.activeValue"
      :inactive-value="info.inactiveValue"
      @change="$emit('change')" />
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
      require: true,
      default: '',
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
      this.$emit('valueChange', val);
    },
    propValue: {
      handler(val) {
        if (val !== this.value) this.value = this.propValue;
      },
      immediate: true,
    },
  },
};
</script>

