<template>
  <kb-row
    class="kb-input-switch"
    type="flex" >
    <label>{{info.label || ''}}</label>
    <kb-switch
      v-model="switchValue"
      :info="switchInfo" />
    <kb-input
      v-model="inputValue"
      :info="inputInfo"
      :disabled="!switchValue"
      type="number" />
  </kb-row>
</template>
<script>
import { isObject } from 'lodash';
import kbRow from '@/components/kbRow.vue';
import kbCol from '@/components/kbCol.vue';
import kbSwitch from '@/components/kbSwitch.vue';
import kbInput from '@/components/kbInput.vue';
import { fSwitch } from '@/utils/formatShortcut';

export default {
  name: 'kbInputSwitch',
  components: {
    kbRow,
    kbSwitch,
    kbInput,
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
  data() {
    return {
      switchInfo: fSwitch('switchValue', '', true, false),
      switchValue: true,
      inputValue: null,
    };
  },
  computed: {
    inputInfo() {
      if (!isObject(this.info)) return {};
      return { ...this.info, label: '' };
    },
  },
  watch: {
    switchValue: {
      handler(newVal) {
        if (newVal === false) this.$emit('valueChange', 0);
        else this.$emit('valueChange', this.inputValue);
      },
      immediate: true,
    },
    inputValue: {
      handler(newVal) {
        if (this.switchValue === false) this.$emit('valueChange', 0);
        else this.$emit('valueChange', newVal);
      },
      immediate: true,
    },
    propValue: {
      handler(newVal) {
        if (newVal !== this.inputValue) {
          this.inputValue = this.propValue;
          if (newVal === 0) this.switchValue = false;
          else this.switchValue = true;
        }
      },
      immediate: true,
    },
  },
};
</script>
<style lang="stylus">
.kb-input-switch
  label
    flex-shrink 0
    text-align right
    width 5.2rem
    margin-right 0.425rem
    line-height 32px
    font-size 0.8rem
  .kb-switch
    flex-basis 0
    margin-right 0.425rem
  .kb-input
    .el-input-group__append
      padding 5px
    input
      padding 0 5px
</style>
