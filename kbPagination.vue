<script>
/**
 * !required!
 * pagination: {
 *   currentPage: Number,
 *   pageSize: Number,
 *   totalSize: Number,
 * }
 */
export default {
  render(h) {
    if (
      this.pagination &&
      this.pagination.totalSize
    ) {
      return h(
        'el-pagination',
        {
          props: {
            background: true,
            layout: this.layout,
            total: this.pagination.totalSize,
            'page-size': this.pagination.pageSize,
            'current-page': this.pagination.currentPage,
            'page-sizes': this.pagination.rowOptions || [10, 20, 50, 100],
          },
          on: {
            'current-change': this.methodUpdateCurrentPage,
            'size-change': this.methodHandleSizeChange,
            'update:currentPage': this.updateHandler,
          },
        },
      );
    }
    return h('div');
  },
  props: {
    pagination: {
      type: Object,
      required: true,
    },
    layout: {
      type: String,
      default: 'total, sizes, prev, pager, next, jumper',
    },
  },
  methods: {
    methodUpdateCurrentPage() {
      // 更新父組件currentPage值
      this.$emit('update:pagination', this.pagination);
      // 觸發current-change事件
      this.$emit('current-change', this.pagination.currentPage);
    },
    updateHandler($event) {
      // 更新當前組件currentPage值
      this.pagination.currentPage = $event;
    },
    methodHandleSizeChange($event) {
      this.pagination.pageSize = $event;
      this.pagination.currentPage = 1;
      this.methodUpdateCurrentPage();
    },
  },
};
</script>

