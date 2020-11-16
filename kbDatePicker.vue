<template>
  <kb-col class="kb-date-picker">
    <template v-if="info.formType">
      <kb-form-item
        :label="info.label"
        :prop="info.validateKey"
        :rules="info.validateRules"
        ref="formItem" >
        <el-date-picker
          v-model="value"
          :type="info.datePickerType"
          :value-format="info.valueFormat"
          :default-time="info.defaultTime"
          :placeholder="info.placeholder || ''"
          @change="methodValueChange($event)"
          v-bind="$attrs"
          size="small" />
      </kb-form-item>
    </template>
    <template v-else>
      <label v-if="info.label">
        {{info.label}}
      </label>
      <el-date-picker
        v-model="value"
        :type="info.datePickerType"
        :value-format="info.valueFormat"
        :default-time="info.defaultTime"
        :placeholder="info.placeholder || ''"
        @change="methodValueChange($event)"
        v-bind="$attrs"
        size="small" />
    </template>
  </kb-col>
</template>
<script>
import kbCol from './kbCol.vue';
import kbFormItem from './kbFormItem.vue';

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
    kbFormItem,
  },
  data() {
    return {
      value: this.propValue,
    };
  },
  watch: {
    propValue(val) {
      if (val !== this.value) {
        this.value = this.propValue;
      }
    },
  },
  methods: {
    methodValueChange(val) {
      if (
        this.info.datePickerType === 'date' &&
        this.info.timeType === 'end' &&
        val !== null
      ) {
        this.$emit('valueChange', val + ((((24*60)*60)*1000)-1));
      } else if (
        this.info.datePickerType === 'datetime' &&
        this.info.defaultTime === '23:59:59' &&
        this.info.valueFormat === 'timestamp' &&
        val) {
        this.$emit('valueChange', val + 999);
      } else {
        this.$emit('valueChange', val);
      }
      if (this.info.formType) {
        this.$refs.formItem.$refs.elFormItem.validate('input');
      }
    },
  },
};
</script>
