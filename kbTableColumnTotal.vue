<template>
  <div>
    <div class="kb-table-column-total" style="white-space: nowrap">
      <template v-if="columnInfo.totalClass">
        <span
          v-if="scope.row[columnInfo.label] || scope.row[columnInfo.label] === 0"
          :class="{
            'kb-table-total-success': isSuccess,
            'kb-table-total-danger': !isSuccess,
          }" >
          {{scope.row[columnInfo.label]}}
        </span>
      </template>
      <template v-else>
        <span
          v-if="scope.row[columnInfo.label] || scope.row[columnInfo.label] === 0" >
          {{scope.row[columnInfo.label]}}
        </span>
      </template>

    </div>
  </div>
</template>
<script>
import { isObject, isBoolean } from 'lodash';

export default {
  props: ['scope', 'columnInfo'],
  computed: {
    isSuccess() {
      if (this.scope.row[this.columnInfo.label] || this.scope.row[this.columnInfo.label] === 0) {
        let tmp = this.scope.row[this.columnInfo.label];
        if (isObject(tmp)) return false;
        else if (isBoolean(tmp)) return false;
        if (typeof tmp === 'string') tmp = Number(tmp.replace(/,/g, ''));
        return this.columnInfo.totalClass ? tmp >= 0 : false;
      }
      return false;
    },
  },
};
</script>

