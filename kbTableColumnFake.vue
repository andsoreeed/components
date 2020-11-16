<template>
  <div class="column-content">
    <div class="column-pre-label" v-if="preLabel">{{preLabel}}</div>
    <div
      v-for="(item, itemIndex) in (columnInfo.group ? columnInfo.group : [columnInfo])"
      :key="`table-column-cell-${itemIndex}`"
      :class="{ 'column-width-100': (item.width || columnInfo.width)
        && (item.type && methodCheckTopStatus(item, scope.row)) }" >
      <template v-if="item.direction === 'column'">
        <div class="kb-table-direction-column">
          <kb-table-column-fake
            v-for="(i, idx) in item.group"
            :key="`kb-table-direction-column-${idx}`"
            :columnInfo="i"
            :scope="scope"
            :preLabel="i.label" />
        </div>
      </template>
      <template v-else-if="item.type && methodCheckTopStatus(item, scope.row)">
        <component
          :is="item.type"
          :item="item"
          :scope="scope" />
      </template>
    </div>
  </div>
</template>
<script>
import { has, isArray, isFunction } from 'lodash';
import kbTableIndex from './kbTableIndex.vue';
import kbTableNormal from './kbTableNormal.vue';
import kbTableTag from './kbTableTag.vue';
import kbTableButton from './kbTableButton.vue';
import kbTablePopover from './kbTablePopover.vue';
import kbTableCheckbox from './kbTableCheckbox.vue';
import kbTableRadioGroup from './kbTableRadioGroup.vue';
import kbTableInput from './kbTableInput.vue';
import kbTableSelect from './kbTableSelect.vue';
import kbTableSwitch from './kbTableSwitch.vue';

export default {
  name: 'kbTableColumnFake',
  props: ['columnInfo', 'scope', 'preLabel'],
  components: {
    kbTableIndex,
    kbTableNormal,
    kbTableTag,
    kbTableButton,
    kbTablePopover,
    kbTableCheckbox,
    kbTableInput,
    kbTableSelect,
    kbTableSwitch,
    kbTableRadioGroup,
  },
  methods: {
    methodCheckTopStatus(item, row) {
      if (has(item, 'topStatusKey') && has(item, 'topStatusValue') && has(item, 'topReverse')) {
        let res;
        const rowValue = row[item.topStatusKey];
        if (isFunction(item.topStatusValue)) res = item.topStatusValue(row);
        else if (isArray(item.topStatusValue)) res = item.topStatusValue.includes(rowValue);
        else res = item.topStatusValue === rowValue;
        return item.topReverse ? !res : res;
      }
      return true;
    },
  },
};
</script>
<style lang="stylus">
.column-content
  display flex
  justify-content center
  flex-wrap nowrap
  align-items center
  .column-pre-label
    font-weight bold
  .column-width-100
    width 100%
.kb-table-direction-column
  .column-content
    &:not(:last-of-type)
      padding-bottom 1px
      margin-bottom 1px
      border-bottom 1px solid #909399
    justify-content left
</style>

