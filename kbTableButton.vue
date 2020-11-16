<template>
  <div class="kb-table-button" :class="`kb-table-button__type-${item.buttonType}`">
    <kb-button
      :buttonName="item.buttonName"
      :buttonBackup="item.buttonBackup"
      :type="item.buttonType"
      :loading="loading"
      @click="methodTablecolumnButton(item, scope.row)" >
      {{handlerDisplayText(scope.row, item.buttonLabel, item.buttonLabelFormat)}}
      <i
        v-if="has(scope.row, item.buttonLabel)"
        class="el-icon-tickets"
        @click.self.stop="copyHandler(scope.row, item.buttonLabel, item.buttonLabelFormat)" />
    </kb-button>
  </div>
</template>
<script>
import { has, isArray, isFunction, isString } from 'lodash';
import { formatNotification } from '../utils/formatFactory';
import kbButton from './kbButton.vue';

export default {
  props: ['item', 'scope'],
  components: {
    kbButton,
  },
  data() {
    return {
      loading: false,
      dialogContentShow: false,
      dialogContentTitle: '',
      dialogContentData: {
        data: '',
      },
    };
  },
  methods: {
    // lodash
    has,
    // method
    async methodTablecolumnButton(item, row) {
      if (item.buttonOverflow) {
        const param = {
          dialogContentShow: true,
          dialogContentData: { data: row[item.buttonLabel] },
          dialogContentTitle: item.label,
        };
        this.$store.commit('global/openTableNormalOverflowPage', param);
        return;
      }
      this.loading = true;
      let initData = null;
      // 判斷是否需要請求初值
      if (item.initGetMethod && isFunction(item.initGetMethod)) {
        if (isArray(item.initGetMethod)) {
          initData = [];
          await item.initGetMethod.forEach(async (getMethod) => {
            const res = await getMethod(row);
            if (!res.code) {
              initData.push(res.data);
            } else {
              formatNotification('提示', '初始化失败', 'error');
            }
            return false;
          });
        } else {
          const res = await item.initGetMethod(row);
          if (has(res, 'code')) {
            if (res.code !== 0) formatNotification('提示', res.msg, 'error');
            else initData = res.data;
          } else {
            initData = res;
          }
        }
      }
      // 判斷是否需要彈窗
      if (item.openDialog && isFunction(item.openDialog)) {
        const res = await item.openDialog(initData, row, item);
        // TODO:
      }
      // TODO:
      // 判斷是否有其他處理
      if (item.otherMethod && isFunction(item.otherMethod)) {
        // TODO:
        const res = await item.otherMethod(initData, row, item);
      }
      this.loading = false;
    },
    handlerDisplayText(row, buttonLabel, method = null) {
      const str = has(row, buttonLabel) ? row[buttonLabel] : buttonLabel;
      return isFunction(method) ? method(str) : str;
    },
    copyHandler(row, buttonLabel, method) {
      let text = this.handlerDisplayText(row, buttonLabel, method);
      const detecNumReg = /^(\d{1,3})(,\d{3})*(,\d{1,3})?\.\d{2}$/;
      if (isString(text) && detecNumReg.test(text)) text = Number(text.replace(/,/g, ''));
      this.$copyText(text).then(() => {
        // 成功
        formatNotification('提示', `<div>复制成功</div><div>${text}</div>`, 'success');
      }, () => {
        // 失败
        formatNotification('提示', '复制失败', 'error');
      });
    },
  },
};
</script>
