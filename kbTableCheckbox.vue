<template>
  <el-checkbox
    v-model="formData.data[scope.$index][item.dataKey]"
    :label="item.showLabel ? item.label : false"
    :disabled="item.disabled"
    :true-label="1"
    :false-label="0"
    @change="handleChange($event, scope.row)" />
</template>
<script>
import { isFunction } from 'lodash';

export default {
  props: ['item', 'scope'],
  inject: {
    formData: {
      default: () => null,
    },
  },
  methods: {
    handleChange(newVal, row) {
      const { item } = this;
      const { methodCB } = item;
      if (methodCB && isFunction(methodCB)) {
        methodCB(newVal, row, item);
      }
    },
  },
};
</script>
