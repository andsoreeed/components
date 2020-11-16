<template>
  <el-table-column
    class="kb-table-index"
    :width="item.width || ''"
    :min-width="item.minWidth || ''"
    align="center" >
    <template slot="header" slot-scope="scope">
      <span :scope="scope">{{item.label}}</span>
      <span
        class="caret-wrapper"
        v-if="item.sortable"
        :class="{ desc: sortableStatus === 'desc', asc: sortableStatus === 'asc' }" >
        <i class="sort-caret ascending" @click="sort('asc', item)()"/>
        <i class="sort-caret descending" @click="sort('desc', item)()"/>
      </span>
    </template>
    <template slot-scope="scope">
      <!-- 小计总计 -->
      <template v-if="scope.row.rowType">
        <kb-table-column-total
          :columnInfo="item"
          :scope="scope" />
      </template>
      <!-- 一般column -->
      <template v-else>
        <kb-table-column-fake
          :columnInfo="item"
          :scope="scope" />
      </template>
    </template>
  </el-table-column>
</template>
<script>
import kbTableColumnFake from './kbTableColumnFake.vue';
import kbTableColumnTotal from './kbTableColumnTotal.vue';

export default {
  props: ['item', 'sortableStatus'],
  components: {
    kbTableColumnFake,
    kbTableColumnTotal,
  },
  methods: {
    sort(str, { sortBy, dataKey }) {
      return () => this.$emit('sortData', [str, sortBy]);
    },
  },
};
</script>
<style lang="stylus">
.el-table
  .caret-wrapper
    margin-left -7px
    cursor auto !important
    .sort-caret
      cursor pointer
    &.asc
      .sort-caret
        &.ascending
          border-bottom-color #44a9c5
    &.desc
      .sort-caret
        &.descending
          border-top-color #44a9c5
</style>

