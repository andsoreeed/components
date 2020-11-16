<script>
/**
  viewDialog - 彈窗區塊組件

  可接收的傳值(從父組件來)
  props: {
    dialogShow: {
      type: Boolean,
      default: false,
      description: 彈窗區塊，控制彈窗顯示狀態。
    },
    dialogTitle:{
      type: String,
      default: '',
      description: 彈窗區塊，彈窗標題
    },
    dialogContent:{
      type: String,
      required: true,
      description: 彈窗區塊，彈窗內容組件路徑
    },
    dialogData {
      type: Object,
      default: {},
      description: 彈窗區塊，要帶進彈窗的主要資料,
    },
    dialogOtherData: {
      type: Object,
      default: null,
      description: 彈窗區塊，要帶進彈窗的其他資料(額外信息)
    },
    dialogWidth: {
      type: String,
      default: '30',
      description: 彈窗區塊，彈窗寬度
    }
  }

  向父層觸發的事件
  $emit: {
    dialogClose: {
      trigger-opportunity: 彈窗顯示狀態由 true 變為 false 時
      param: void,
    }
  }

  向彈窗內容組件(子模板組件)傳值
  To children component props: {
    dialogContentData: {
      type: Object,
      description: 彈窗區塊，從頁面帶進彈窗的主要資料，直接傳至子模板組件
    },
    dialogOtherData: {
      type: Object,
      description: 彈窗區塊，從頁面帶進彈窗的其他資料，直接傳至子模板組件
    }
  },

  向彈窗內容組件(子模板組件)綁定事件
  To children component on: {
    dialogClose: {
      param: void,
      description: 關閉彈窗
    }
  }

  對彈窗內容組件(子模板組件)預設執行的方法(如果子組件有的話)
  To children component methods: {
    destroy,
  }
 */
import { has, isFunction } from 'lodash';

export default {
  render(h) {
    return h(
      'div',
      {
        class: {
          kbViewDialog: true,
        },
      },
      [
        h(
          'el-dialog',
          {
            props: {
              title: this.dialogTitle,
              visible: this.localDialogShow,
              width: `${this.dialogWidth}%`,
              appendToBody: this.appendToBody,
              modal: this.modal,
              center: this.center,
            },
            on: {
              'update:visible': ($event) => {
                this.localDialogShow = $event;
              },
            },
          },
          [
            h(require(`@/${this.dialogContent}`).default, {
              props: {
                dialogShow: this.localDialogShow,
                dialogContentData: this.dialogData,
                dialogOtherData: this.dialogOtherData,
              },
              on: {
                dialogClose: this.methodDialogClose,
              },
              ref: 'dialog',
            }),
          ],
        ),
      ],
    );
  },
  props: {
    dialogShow: {
      type: Boolean,
      default: () => false,
    },
    dialogTitle: {
      type: String,
      default: () => '',
    },
    dialogContent: {
      type: String,
      required: true,
    },
    dialogData: {
      type: [Object, Array],
      default: () => { },
    },
    dialogOtherData: {
      default: () => null,
    },
    dialogWidth: {
      type: String,
      default: () => '30',
    },
    appendToBody: {
      type: Boolean,
      default: () => false,
    },
    modal: {
      type: Boolean,
      default: () => true,
    },
    center: {
      type: Boolean,
      default: () => false,
    },
  },
  data() {
    return {
      localDialogShow: this.dialogShow,
    };
  },
  watch: {
    dialogShow(val) {
      if (!val) {
        // 彈窗關閉
        this.$emit('dialogClose');
        if (
          has(this.$refs.dialog, 'destroy') &&
          isFunction(this.$refs.dialog.destroy)
        ) {
          this.$refs.dialog.destroy();
        }
      }
      this.localDialogShow = val;
    },
    localDialogShow(val) {
      this.methodUpdateDialogShow();
    },
  },
  methods: {
    methodDialogClose() {
      this.localDialogShow = false;
    },
    methodUpdateDialogShow() {
      this.$emit('update:dialogShow', this.localDialogShow);
    },
  },
};
</script>
