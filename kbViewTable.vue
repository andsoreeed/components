<template>
  <div class="kbViewTable">
    <kb-card class="box-card">
      <div class="card-title" v-if="tableTitle || tableTitleMethod">
        <div v-if="tableTitle">{{tableTitle}}</div>
        <div style="margin-left: auto">
          <kb-button
            v-if="tableTitleMethod"
            type="primary"
            :loading="tableTitleMethodLoading"
            @click="handleTableTitleMethod(tableTitleMethod)" >
              {{tableTitleButtenText}}
          </kb-button>
        </div>
      </div>
      <kb-table
        :tableData="localTableTotalData"
        :tableInfo="tableInfo"
        :tableSpanMethod="tableSpanMethod"
        @selection-change="handleSelectionChange"
        @expand-change="handleExpandChange"
        :handlerRowClassName="handlerRowClassName"
        :tableEmptyStr="tableEmptyStr"
        @sortData="sortData"
        :sortableData="sortableData"
        ref="kbTable" />
      <kb-pagination
        v-if="localTablePagination"
        :pagination="localTablePagination"
        @current-change="methodUpdateCurrentPage" />
    </kb-card>
    <kb-view-dialog
      :dialogShow.sync="dialogContentShow"
      :dialogTitle="dialogContentTitle"
      dialogContent="components/kbTableNormalOverflowPage"
      :dialogData="dialogContentData"
      dialogWidth="30" />
  </div>
</template>
<script>
/**
  viewTalbe - 表格區塊組件

  可接收的傳值(從父組件來)
  props: {
    tableInfo: {
      type: Array,
      required: true,
      description: 表格區塊，表格內容配置陣列。
    },
    tableData:{
      type: Array,
      description: 表格區塊，要顯示在表格內的資料陣列
    },
    tablePagination:{
      type: Object,
      description: 表格區塊，分頁組件配置項,
    },
    tableTitle: {
      type: String,
      description: 表格區塊，表格標題,
    },
  }
 */
import { cloneDeep, sortBy } from 'lodash';
import kbCard from './kbCard.vue';
import kbTable from './kbTable.vue';
import kbPagination from './kbPagination.vue';
import kbViewDialog from './kbViewDialog.vue';
import kbButton from './kbButton.vue';

export default {
  name: 'kbViewTable',
  props: {
    tableInfo: {
      type: [Array, Promise],
      required: true,
    },
    tableData: {
      type: Array,
    },
    tablePagination: {
      type: Object,
    },
    tableTitle: {
      type: String,
    },
    tableTitleMethod: {
      type: [Function, Boolean],
    },
    tableTitleButtenText: {
      type: String,
    },
    tableTotalData: {
      type: Array,
    },
    tableSpanMethod: {
      type: Function,
    },
    tableEmptyStr: {
      type: String,
      default: '',
    },
    handlerRowClassName: {
      type: Function,
    },
  },
  components: {
    kbCard,
    kbTable,
    kbPagination,
    kbViewDialog,
    kbButton,
  },
  data() {
    return {
      localTablePagination: cloneDeep(this.tablePagination),
      localTableInfo: [],
      localTableData: [],
      dialogContentShow: false,
      dialogContentData: {
        data: '',
      },
      dialogContentTitle: '',
      tableTitleMethodLoading: false,
      sortableData: {
        sortableIdx: null,
        sortableStatus: '',
        sortableDataKey: '',
      },
    };
  },
  computed: {
    tableNormalOverflowPageStatus() {
      return this.$store.getters['global/tableNormalOverflowPageStatus'];
    },
    localTableTotalData() {
      const tmp = cloneDeep(this.localTableData);
      if (
        tmp &&
        tmp.length &&
        this.tableTotalData &&
        this.tableTotalData.length
      ) {
        this.tableTotalData.forEach((item) => {
          const dataKeyObject = {};
          item.dataKey.forEach((dataKey, dataKeyIndex) => {
            dataKeyObject[dataKey] = item.data[dataKeyIndex];
          });
          if (this.localTableInfo[0]) {
            tmp.push({
              ...dataKeyObject,
              [`${this.localTableInfo[0].label}`]: item.label,
              rowType: 'tableTotal',
            });
          }
        });
      }
      return tmp;
    },
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
    tableNormalOverflowPageStatus(newVal) {
      if (newVal) {
        const { tableNormalOverflowPage: dialogInfo } = this.$store.state.global;
        if (this.$parent.$el.offsetParent === null) return;
        const { dialogContentData, dialogContentTitle } = dialogInfo.data;
        this.dialogContentData = dialogContentData;
        this.dialogContentTitle = dialogContentTitle;
        this.dialogContentShow = true;
      }
    },
    dialogContentShow(newVal) {
      if (!newVal) {
        this.$store.commit('global/closeTableNormalOverflowPage');
        this.dialogContentData.data = '';
        this.dialogContentTitle = '';
      }
    },
    tablePagination: {
      handler() {
        this.localTablePagination = cloneDeep(this.tablePagination);
      },
      deep: true,
    },
    tableInfo(newVal) {
      if (newVal.length) {
        this.localTableInfo = sortBy(
          cloneDeep(this.tableInfo),
          value => value.sort,
        );
      }
    },
    tableData: {
      handler(newVal) {
        this.$nextTick(() => {
          if (this.sortableData.sortableStatus) this.sortData(this.sortableData, false);
          else this.localTableData = cloneDeep(newVal);
        });
      },
      immediate: true,
    },
  },
  methods: {
    methodUpdateCurrentPage() {
      this.$emit('update:tablePagination', this.localTablePagination);
      this.$emit('pagination-change');
    },
    handleSelectionChange(selection) {
      this.$emit('selection-change', selection);
    },
    handleExpandChange(...expand) {
      this.$emit('expand-change', ...expand);
    },
    toggleRowExpansion(...params) {
      this.$refs.kbTable.toggleRowExpansion(...params);
    },
    async handleTableTitleMethod(method) {
      this.tableTitleMethodLoading = true;
      await method();
      this.tableTitleMethodLoading = false;
    },
    sortData(data, status = true) {
      const { sortableIdx, sortableStatus, sortableDataKey } = data;
      const tableData = cloneDeep(this.tableData);
      if (sortableStatus === 'asc') {
        this.localTableData = sortBy(tableData, [o => o[sortableDataKey] || 0]);
      } else if (sortableStatus === 'desc') {
        this.localTableData = sortBy(tableData, [o => o[sortableDataKey] || 0]).reverse();
      } else this.localTableData = tableData;
      if (status) this.sortableData = { sortableIdx, sortableStatus, sortableDataKey };
    },
  },
};
</script>

