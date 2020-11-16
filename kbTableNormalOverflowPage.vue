<template>
  <div class="msg-content">
    <kb-row type="flex" justify="space-around">
      <kb-col class="msg-content--title">
        {{dialogContentData.data}}
      </kb-col>
    </kb-row>
    <div class="dialog-footer">
      <kb-button
        type="primary"
        icon="el-icon-close"
        @click="methodCloseDialog()" >
        关闭
      </kb-button>
    </div>
  </div>
</template>
<script>
import kbRow from '@/components/kbRow.vue';
import kbCol from '@/components/kbCol.vue';
import kbButton from '@/components/kbButton.vue';

export default {
  props: ['dialogContentData'],
  components: {
    kbButton,
    kbCol,
    kbRow,
  },
  methods: {
    methodCloseDialog() {
      this.$emit('dialogClose');
    },
  },
};
</script>

<style lang="stylus">
.msg-content
  white-space normal
  text-align left
  .dialog-footer
    .kb-button
      width auto
      display: inline-flex;
      align-items center
      span
        display inline-block
</style>
