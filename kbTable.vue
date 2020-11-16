<template>
  <el-table
    class="kb-table"
    :data="tableData"
    :span-method="tableSpanMethod"
    :show-header="tableShowHeader"
    @selection-change="handleSelectionChange"
    @expand-change="handleExpandChange"
    :row-class-name="isFunction(handlerRowClassName) ? handlerRowClassName : null"
    stripe
    ref="elTable" >
    <component
      v-for="(item, indexInfo) in localTableInfo"
      :key="`table-column-${indexInfo}`"
      :is="specifyType.includes(item.type) ? item.type : 'kb-table-column'"
      :item="item"
      @sortData="sortData($event, indexInfo)"
      :sortableStatus="indexInfo === sortableData.sortableIdx && sortableData.sortableStatus" />
    <div slot="empty">{{`${tableEmptyStr ? tableEmptyStr : '暂无数据'}`}}</div>
  </el-table>
</template>
<script>
import { cloneDeep, sortBy, isFunction, isArray } from 'lodash';
import kbTableSub from './kbTableSub.vue';
import kbTableColumn from './kbTableColumn.vue';
import kbTableColumnSelection from './kbTableColumnSelection.vue';
import kbTableColumnExpand from './kbTableColumnExpand.vue';

export default {
  props: {
    tableData: {
      type: Array,
    },
    tableInfo: {
      type: [Array, Promise],
      required: true,
    },
    tableSpanMethod: {
      type: Function,
    },
    tableEmptyStr: {
      type: String,
      default: '',
    },
    tableShowHeader: {
      type: Boolean,
      default: true,
    },
    handlerRowClassName: {
      type: Function,
    },
    sortableData: {
      type: Object,
      default: () => ({ sortableIdx: null, sortableStatus: '', sortableDataKey: '' }),
    },
  },
  components: {
    kbTableSub,
    kbTableColumn,
    kbTableColumnSelection,
    kbTableColumnExpand,
  },
  data() {
    return {
      specifyType: [
        'kb-table-sub',
        'kb-table-column-selection',
        'kb-table-column-expand',
      ],
      localTableInfo: [],
    };
  },
  created() {
    Promise.resolve(this.tableInfo).then((val) => {
      this.localTableInfo = sortBy(
        cloneDeep(val),
        value => value.sort,
      );
    });
  },
  watch: {
    tableInfo(newVal) {
      if (newVal.length) {
        this.localTableInfo = sortBy(
          cloneDeep(this.tableInfo),
          value => value.sort,
        );
      }
    },
    tableData: {
      handler() {
        this.$nextTick(() => {
          setTimeout(() => {
            if (isArray(this.localTableInfo) &&
            isArray(this.tableData)) {
              const columnLen = this.localTableInfo.length;
              const columnLenReal = this.localTableInfo.reduce((arr, cur) => {
                if (isArray(cur.subInfo)) return (arr + cur.subInfo.length);
                return arr + 1;
              }, 0);
              const dataLen = this.tableData.length;
              const tableColumn = this.$children[0].$children[1].$children;
              const subColumnIndexRecord = new Map();
              this.localTableInfo.forEach((item, iColumn) => {
                const subStatus = isArray(item.subInfo);
                const subIndex = subColumnIndexRecord.has(iColumn) ?
                  subColumnIndexRecord.get(iColumn) :
                  0;
                const columnWrapper = subStatus ? item.subInfo : [item];
                subColumnIndexRecord.set(
                  iColumn + 1,
                  subIndex + (subStatus ? item.subInfo.length - 1 : 0),
                );
                columnWrapper.forEach((subItem, subIColumn) => {
                  let maxWidth = 0;
                  if (subItem.label && subItem.label.length) {
                    const strArr = subItem.label.split('');
                    strArr.forEach((str) => {
                      if (/[\u4e00-\u9fa5]/.test(str)) maxWidth += 15;
                      else maxWidth += 5;
                    });
                  }
                  if (subItem.sortable) maxWidth += 10;

                  maxWidth = this.calCellWidth(
                    0,
                    dataLen,
                    columnLenReal,
                    tableColumn,
                    iColumn + subIColumn + subIndex,
                    maxWidth,
                  );
                  if (maxWidth) {
                    const outputWidth = (maxWidth + 8).toString();
                    if (!subItem.minWidth) {
                      this.$set(subItem, 'minWidth', outputWidth);
                    } else {
                      subItem.minWidth = outputWidth;
                    }
                  }
                });
              });
            }
          }, 0);
        });
      },
      immediate: true,
    },
  },
  methods: {
    isFunction,
    handleSelectionChange(selection) {
      this.$emit('selection-change', selection);
    },
    handleExpandChange(...expand) {
      this.$emit('expand-change', ...expand);
    },
    calCellWidth(iData, dataLen, columnReal, columnAll, columnIndex, width) {
      const cell = columnAll[columnIndex + 1 + (iData * columnReal)];
      let cellWidth = 0;
      if (cell) {
        let haveKbTableButton = false;
        const nodesLen = cell.$el.childNodes.length;
        cell.$el.childNodes.forEach((c, cIndex) => {
          if (c.nodeName !== '#comment'
            && c.firstChild
            && c.firstChild.nodeName !== '#comment') {
            const element = c.firstChild;
            let offsetWidth = 0;
            if (/kb-input/.test(c.className)) offsetWidth = 0;
            else if (/kb-table-normal/.test(c.className) && cIndex + 1 !== nodesLen) offsetWidth = 0;
            else if (/el-button--text/.test(element.className)) {
              const strArr = element.innerText.split('').slice(0, -1);
              strArr.forEach((str) => {
                offsetWidth += (/[\u4e00-\u9fa5]/.test(str) ? 15 : 5);
              });
            } else if (element.offsetWidth === undefined) ({ offsetWidth } = c);
            else ({ offsetWidth } = element);
            if (/kb-table-button/.test(c.classList) && !haveKbTableButton) haveKbTableButton = true;
            cellWidth += Number(offsetWidth);
          }
        });
        cellWidth += haveKbTableButton ? 30 : 0;
        // 調適用，不刪
        if (columnIndex === 1) {
          // e(cell, cellWidth, width);
        }
        if (iData < dataLen) {
          return this.calCellWidth(
            iData + 1,
            dataLen,
            columnReal,
            columnAll,
            columnIndex,
            Math.max(width, cellWidth),
          );
        }
      }
      return Math.max(width, cellWidth);
    },
    toggleRowExpansion(...params) {
      this.$refs.elTable.toggleRowExpansion(...params);
    },
    toggleRowSelection(row, selected) {
      this.$refs.elTable.toggleRowSelection(row, selected);
    },
    clearSelection() {
      this.$refs.elTable.clearSelection();
    },
    sortData(data, idx) {
      const { sortableIdx, sortableStatus } = this.sortableData;
      const [str, dataSortBy] = data;
      const status = sortableIdx === idx && sortableStatus === str;
      const sortableData = status
        ? { sortableIdx: null, sortableStatus: '', sortableDataKey: '' }
        : { sortableIdx: idx, sortableStatus: str, sortableDataKey: dataSortBy };
      this.$emit('sortData', sortableData);
    },
  },
};
</script>

