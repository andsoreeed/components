<template>
  <div class="kb-table-tag">
    <el-tag
      :type="item.value" >
      {{handleFormat(scope.row[item.tagLabel] !== undefined ?
        scope.row[item.tagLabel] :
        item.tagLabel)}}
    </el-tag>
  </div>
</template>
<script>
import { isFunction } from 'lodash';

export default {
  props: ['item', 'scope'],
  methods: {
    handleFormat(str) {
      if (isFunction(this.item.methodFormat)) return this.item.methodFormat(str);
      return str;
    },
  },
};
</script>
