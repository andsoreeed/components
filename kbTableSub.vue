<template>
  <el-table-column
    class="kb-table-Sub"
    :label="item.label"
    :width="item.width || ''"
    :min-width="item.minWidth || ''"
    align="center" >
    <component
      v-for="(subItem, subIndex) in item.subInfo"
      :key="`table-column-sub-${subIndex}`"
      :is="specifyType.includes(subItem.type) ? subItem.type : 'kb-table-column'"
      :item="subItem" />
  </el-table-column>
</template>
<script>
import kbTableColumn from './kbTableColumn.vue';
import kbTableColumnSelection from './kbTableColumnSelection.vue';

export default {
  props: ['item'],
  components: {
    kbTableColumn,
    kbTableColumnSelection,
  },
  data() {
    return {
      specifyType: ['kb-table-column-selection'],
    };
  },
};
</script>
