<template>
  <el-popover
    class="kb-table-popover"
    :placement="item.placement || 'bottom-start'"
    :title="item.title"
    :width="item.popoverWidth"
    :trigger="item.trigger" >
    <view-popover
      :data="item"
      :rowData="scope.row" />
    <kb-button
      slot="reference"
      type="text" >
      {{handlerDisplayText(scope.row, item.content, item.method)}}
      <i
        v-if="has(scope.row, item.content)"
        class="el-icon-tickets"
        @click.self.stop="copyHandler(scope.row, item.content)" />
    </kb-button>
  </el-popover>
</template>
<script>
import { has } from 'lodash';
import viewPopover from './kbViewPopover.vue';
import kbButton from './kbButton.vue';
import { formatNotification } from '../utils/formatFactory';

export default {
  props: ['item', 'scope'],
  components: {
    viewPopover,
    kbButton,
  },
  methods: {
    has,
    handlerDisplayText(row, content, method) {
      const result = has(row, content) ? row[content] : content;
      return method ? method(result) : result;
    },
    copyHandler(row, content) {
      const text = this.handlerDisplayText(row, content);
      const num = Number(text.replace(/,/g, ''));
      this.$copyText(num).then(() => {
        // 成功
        formatNotification('提示', `<div>复制成功</div><div>${num}</div>`, 'success');
      }, () => {
        // 失败
        formatNotification('提示', '复制失败', 'error');
      });
    },
  },
};
</script>
