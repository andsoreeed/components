<template>
  <el-table-column
    class="kb-table-column-expand"
    type="expand">
    <template slot-scope="props">
      <component
        v-if="item.component"
        :is="item.component"
        :props="props.row.expand"
        :expandFormat="item.expandFormat"
        :otherData="item.otherData" />
    </template>
  </el-table-column>
</template>
<script>
export default {
  name: 'kbTableColumnExpand',
  props: ['item'],
};
</script>

