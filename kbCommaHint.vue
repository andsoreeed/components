<template>
  <kb-row class="kb-comma-hint">
    <kb-col>
      * 多用户请使用半角逗点隔开 范例： user1,user2,user3
      <kb-button
        type="primary"
        @click="copyComma">复制逗号</kb-button>
      </kb-col>
  </kb-row>
</template>
<script>
import kbRow from '@/components/kbRow.vue';
import kbCol from '@/components/kbCol.vue';
import kbButton from '@/components/kbButton.vue';
import globalMixin from '@/mixins/globalMixin';
import { fNotify } from '@/utils/formatShortcut';

export default {
  name: 'kbCommaHint',
  components: {
    kbRow, kbCol, kbButton,
  },
  mixins: [globalMixin],
  methods: {
    copyComma() {
      const text = ',';
      this.$copyText(text).then(() => {
        // 成功
        fNotify('提示', `<div>复制成功</div><div>${text}</div>`, 'success');
      }, () => {
        // 失败
        fNotify('提示', '复制失败', 'error');
      });
    },
  },
};
</script>
<style lang="stylus">
.kb-comma-hint
  & > div
    margin-left 5.3rem
</style>

