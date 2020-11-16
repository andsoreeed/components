<template>
  <div class="search-group"
    @mouseenter="handleMouseenter()"
    @mouseleave="handleMouseleave()"
    :style="{ marginBottom: `${styleMarginBottom}px`, transition: 'margin .2s ease'}"
    ref="kb-search-group" >
    <kb-card class="box-card">
      <kb-form :model="queryParams" ref="kbForm">
        <kb-row
          type="flex"
          justify="start" >
          <kb-col
            :span="6"
            v-for="(inputItem, index) in localSearchInputGroup"
            :key="index" >
            <component
              v-if="inputItem.type !== 'block'"
              :is="inputItem.type"
              v-model="queryParams[inputItem.dataKey]"
              :disabled="inputItem.disabled"
              :info="inputItem"
              @keyup.enter.native="methodKeyupSearch()" />
          </kb-col>
          <kb-row class="search-group-button" type="flex" justify="end" style="flex: 1">
            <div
              v-for="(buttonItem, index) in localSearchButtonGroup"
              :key="index" >
              <template v-if="buttonItem.type === 'button-search'">
                <kb-button
                  ref="searchButton"
                  type="primary"
                  icon="el-icon-search"
                  :loading="searchLoading"
                  @click="methodSearch(buttonItem.method)" >
                  {{buttonItem.label}}
                </kb-button>
              </template>
              <template v-else-if="buttonItem.type === 'button-reset'">
                <kb-button
                  type="text"
                  @click="methodReset(buttonItem.method)" >
                  重置
                </kb-button>
              </template>
              <template v-else-if="buttonItem.type === 'button-custom'">
                <kb-button
                  :type="buttonItem.buttonType"
                  :icon="buttonItem.icon"
                  :buttonName="buttonItem.buttonName"
                  :buttonBackup="buttonItem.buttonBackup"
                  @click="methodCustomButtonMethod(buttonItem)" >
                  {{buttonItem.label}}
                </kb-button>
              </template>
              <template v-else-if="buttonItem.type === 'button-shortcut-date'">
                <kb-row type="flex">
                  <kb-button-group>
                    <kb-button
                      v-for="(shortcutDate, shortcutDateIndex) in buttonShortcutDate"
                      :key="`button-shortcut-date-${shortcutDateIndex}`"
                      type="primary"
                      @click="methodShortcutDate(shortcutDate.dataKey)">
                      {{shortcutDate.label}}
                    </kb-button>
                  </kb-button-group>
                </kb-row>
              </template>
            </div>
          </kb-row>
        </kb-row>
      </kb-form>
    </kb-card>
  </div>
</template>
<script>
import { has, isEmpty, forOwn, cloneDeep, sortBy, isArray, isString, trim } from 'lodash';
import NProgress from 'nprogress';
import { Notification } from 'element-ui';
import kbInput from './kbInput.vue';
import kbSelect from './kbSelect.vue';
import kbSelectLottery from './kbSelectLottery.vue';
import kbDatePicker from './kbDatePicker.vue';
import kbButton from './kbButton.vue';
import kbRow from './kbRow.vue';
import kbCol from './kbCol.vue';
import kbCard from './kbCard.vue';
import kbTabsGroup from './kbTabsGroup.vue';
import kbForm from './kbForm.vue';
import kbButtonGroup from './kbButtonGroup.vue';
import getDateDay from '../utils/getDateDay';
import { formatPagination } from '../utils/formatFactory';

/**
  searchGroup - 搜索區塊組件

  可接收的傳值(從父組件來)
  props: {
    searchInputInitGroup: {
      type: Object,
      default: {},
      description: 搜索區塊，搜索控件組預設值。其中key為dataKey, value為預設值
    },
    searchInputGroup:{
      type: Array,
      description: 搜索區塊，搜索控件組配置。配置可通過formatSearchFactory中的function拿到
    },
    searchButtonGroup:{
      type: Array,
      description: 搜索區塊，搜索按鈕組配置,
    },
    searchPagination: {
      type: Number,
      description: 搜索區塊，搜索頁數,
    },
  }

  向父層觸發的事件
  $emit: {
    searchEnd: {
      trigger-opportunity: 搜索按鈕發出搜索請求，並請求成功拿到搜索結果時
      param: res.data,
      param-description: 搜索結果
    }
  }

  向父層開放的方法
  methods: {
    methodResearch: {
      param: void,
      description: 觸發搜索動作
    },
    methodReset: {
      param: Function,
      param-description: 重置後callback
      description: 觸發重置動作
    }
  }
 */
export default {
  props: {
    searchInputInitGroup: {
      type: Object,
      default: () => { },
    },
    searchInputGroup: {
      type: [Array, Promise],
    },
    searchButtonGroup: {
      type: [Array, Promise],
    },
    searchPagination: {
      type: Object,
    },
    dateValidateThreeMonth: {
      type: Boolean,
      default: true,
    },
  },
  components: {
    kbInput,
    kbSelect,
    kbSelectLottery,
    kbDatePicker,
    kbButton,
    kbRow,
    kbCard,
    kbTabsGroup,
    kbForm,
    kbCol,
    kbButtonGroup,
  },
  data() {
    return {
      stopResearch: false,
      localSearchPagination: this.searchPagination
        ? cloneDeep(this.searchPagination)
        : formatPagination(),
      queryParams: {},
      localSearchInputGroup: [],
      localSearchButtonGroup: [],
      searchLoading: false,
      exportLoading: false,
      buttonShortcutDate: [
        { label: '今日', dataKey: 'today' },
        { label: '昨日', dataKey: 'yesterday' },
        { label: '本周', dataKey: 'week' },
        { label: '上周', dataKey: 'lastweek' },
        { label: '本月', dataKey: 'month' },
        { label: '上月', dataKey: 'lastmonth' },
      ],
      shortcutDateStartProperty: '',
      shortcutDateEndProperty: '',
      styleMarginBottom: 0,
      setTimeoutId: null,
      // 搜索欄伸縮開關
      // TODO:
      switchFlexible: false,
    };
  },
  created() {
    Promise.resolve(this.searchButtonGroup).then((searchButtonGroupList) => {
      this.localSearchButtonGroup = cloneDeep(searchButtonGroupList);
    });
    Promise.resolve(this.searchInputGroup).then((searchInputGroup) => {
      if (searchInputGroup.length) {
        const queryParams = {};
        this.localSearchInputGroup = sortBy(cloneDeep(searchInputGroup), value => value.sort);
        this.localSearchInputGroup.forEach((item) => {
          if (item.dataKey) {
            queryParams[item.dataKey] = has(this.searchInputInitGroup, item.dataKey) ?
              this.searchInputInitGroup[item.dataKey] :
              '';
            if (item.timeType) {
              if (item.timeType === 'start') {
                this.shortcutDateStartProperty = item.dataKey;
              } else if (item.timeType === 'end') {
                this.shortcutDateEndProperty = item.dataKey;
              }
            }
          }
        });
        this.queryParams = cloneDeep(queryParams);
      }
    });
  },
  watch: {
    searchPagination: {
      handler(newVal, oldVal) {
        this.$nextTick(() => {
          this.localSearchPagination = cloneDeep(this.searchPagination);
          if ((newVal.currentPage === oldVal.currentPage) &&
            (newVal.pageSize === oldVal.pageSize)) return;
          if (this.stopResearch) {
            this.stopResearch = false;
          } else {
            this.methodResearch();
          }
        });
      },
      deept: true,
    },
    searchButtonGroup: {
      handler(newVal) {
        Promise.resolve(newVal).then((searchButtonGroup) => {
          this.localSearchButtonGroup = cloneDeep(searchButtonGroup);
        });
      },
      deep: true,
    },
    searchInputGroup: {
      handler(newVal) {
        Promise.resolve(newVal).then((searchInputGroup) => {
          if (searchInputGroup.length === 0) {
            this.localSearchInputGroup.splice(0);
            return;
          }
          const status = searchInputGroup.length &&
            this.localSearchInputGroup.length &&
            searchInputGroup.length < this.localSearchInputGroup.length;
          if (status) this.localSearchInputGroup.splice(searchInputGroup.length);
          sortBy(searchInputGroup, value => value.sort).forEach((item, index) => {
            const multiple = item.type === 'kb-select' && item.multiple;
            // 動態更換新配置
            // 檢查dataKey，判斷queryParam需不需要做修正
            if (!has(this.queryParams, item.dataKey)) {
              let initVal = '';
              if (has(this.searchInputInitGroup, item.dataKey)) {
                initVal = this.searchInputInitGroup[item.dataKey];
              } else if (multiple) initVal = [];
              this.$set(
                this.queryParams,
                item.dataKey,
                initVal,
              );
            }
            // 更新 shortcutDateProperty
            if (item.timeType) {
              if (item.timeType === 'start') {
                this.shortcutDateStartProperty = item.dataKey;
              } else if (item.timeType === 'end') {
                this.shortcutDateEndProperty = item.dataKey;
              }
            }
            // 多餘舊配置的項
            if (!status && index >= this.localSearchInputGroup.length) {
              this.localSearchInputGroup.push(item);
            }
            // 位置相同需要動態更換的項
            if (item.dynamicChange) {
              this.localSearchInputGroup.splice(index, 1, item);
            }
            // select框，options與當前value的對應檢查
            if (item.type === 'kb-select' && item.checkOptions) {
              if (isArray(this.localSearchInputGroup[index].options)) {
                let checkStatus = false;
                let checkCount = 0;
                this.localSearchInputGroup[index].options.forEach((optionItem) => {
                  // 多選框
                  if (item.multiple && this.queryParams[item.dataKey].includes(optionItem.id)) {
                    checkStatus = true;
                    checkCount += 1;
                    // 單選框
                  } else if (optionItem.id === this.queryParams[item.dataKey]) {
                    checkStatus = true;
                  }
                });
                if (isArray(this.queryParams[item.dataKey])) {
                  if (checkCount === this.queryParams[item.dataKey].length) checkStatus = true;
                  else checkStatus = false;
                }
                if (!checkStatus) {
                  if (isArray(this.queryParams[item.dataKey])) {
                    this.queryParams[item.dataKey].splice(0);
                  } else this.queryParams[item.dataKey] = '';
                }
              }
            }
          });
        });
      },
      deep: true,
    },
    searchInputInitGroup: {
      handler() {
        if (!isEmpty(this.searchInputInitGroup)) {
          forOwn(this.searchInputInitGroup, (value, key) => {
            if (has(this.queryParams, key)) {
              this.queryParams[key] = value;
            }
          });
        }
      },
      deep: true,
    },
    queryParams: {
      handler(newVal, oldVal) {
        if (!isEmpty(oldVal) && this.localSearchPagination.currentPage !== 1) {
          this.stopResearch = true;
          this.localSearchPagination.currentPage = 1;
          this.$emit('update:searchPagination', this.localSearchPagination);
        }
      },
      deep: true,
    },
  },
  methods: {
    handleMouseenter() {
      if (!this.switchFlexible) return;
      this.styleMarginBottom = 0;
      clearTimeout(this.setTimeoutId);
      this.setTimeoutId = null;
    },
    handleMouseleave() {
      if (!this.switchFlexible) return;
      if (this.setTimeoutId) {
        clearTimeout(this.setTimeoutId);
        this.setTimeoutId = null;
      }
      const dropdowns = document.querySelectorAll('.el-select-dropdown, .el-date-picker');
      const dropdownHasOpen = new Set([...dropdowns].map(item => item.style.display));
      if (dropdownHasOpen.has('') || dropdownHasOpen.has('block')) return;

      this.setTimeoutId = setTimeout(() => {
        const element = this.$refs['kb-search-group'];
        const { offsetHeight } = element;
        if (offsetHeight < 65) return;
        this.styleMarginBottom = -1 * (offsetHeight - 60);
      }, 1000);
    },
    // 對外開放方法，重新搜索
    methodResearch() {
      setTimeout(() => {
        this.$refs.searchButton[0].$el.click();
      }, this.$refs.searchButton ? 0 : 200);
    },
    // 對外開放方法，重置搜索
    methodReset(callback) {
      forOwn(this.queryParams, (value, key) => {
        if (has(this.searchInputInitGroup, key)) {
          this.queryParams[key] = this.searchInputInitGroup[key];
        } else if (isArray(this.queryParams[key])) this.queryParams[key] = [];
        else this.queryParams[key] = '';
      });
      this.$refs.kbForm.resetFields();
      if (callback) {
        callback();
      }
    },
    methodOutputParam() {
      return this.methodInitOutputQueryParams(this.queryParams);
    },
    methodKeyupSearch() {
      if (this.$refs.searchButton) this.methodResearch();
    },
    methodNotify(title, message, type = 'success') {
      Notification({
        title,
        message,
        type,
        duration: 2000,
        position: 'top-right',
      });
    },
    async methodCustomButtonMethod(item) {
      let initData = null;
      if (item.initGetMethod) {
        if (isArray(item.initGetMethod)) {
          initData = [];
          const initGetMethod = [];
          item.initGetMethod.forEach((getMethod) => {
            initGetMethod.push(new Promise(async (resolve, reject) => {
              const res = await getMethod(this.methodInitOutputQueryParams(this.queryParams));
              if (!res.code) {
                resolve(res);
              } else {
                this.methodNotify('提示', '初始化失败', 'error');
                reject(res);
              }
            }));
          });
          const res = await Promise.all(initGetMethod);
          res.forEach((resItem) => {
            if (!resItem.code) initData.push(resItem.data);
          });
        } else {
          const res = await item.initGetMethod(this.methodInitOutputQueryParams(this.queryParams));
          // TODO:
          if (has(res, 'code')) {
            if (!res.code) {
              initData = res.data;
            }
          }
          initData = res;
        }
      }
      if (item.method) {
        item.method(initData, this.methodInitOutputQueryParams(this.queryParams), item);
      }
    },
    methodShortcutDate(dayType) {
      setTimeout(() => {
        let [startTime, endTime] = [null, null];
        if (this.shortcutDateStartProperty) startTime = getDateDay(dayType, 'start');
        if (this.shortcutDateEndProperty) endTime = getDateDay(dayType, 'end');
        ([this.queryParams[this.shortcutDateStartProperty],
          this.queryParams[this.shortcutDateEndProperty]] = [startTime, endTime]);
        this.$emit('updateShortcutDate', [startTime, endTime]);
      }, this.shortcutDateStartProperty && this.shortcutDateEndProperty ? 0 : 200);
    },
    methodInitOutputQueryParams(obj) {
      // 初始化請求物件
      const queryParams = cloneDeep(obj);
      forOwn(queryParams, (value, key) => {
        if (isString(value)) {
          queryParams[key] = trim(value);
        }
        if (value === '' || value === -1) {
          queryParams[key] = null;
        }
      });
      return {
        ...queryParams,
      };
    },
    methodDateValidate() {
      const [dateStartKey, dateEndKey] =
        [this.shortcutDateStartProperty, this.shortcutDateEndProperty];
      // 結束時間不小於開始時間
      if (dateStartKey && dateEndKey &&
        this.queryParams[dateStartKey] &&
        this.queryParams[dateEndKey] &&
        (this.queryParams[dateEndKey] - this.queryParams[dateStartKey]) < 0) {
        this.methodNotify('提示', '结束时间不能早于开始时间', 'warning');
        return false;
      }
      if (this.dateValidateThreeMonth && dateStartKey && this.queryParams[dateStartKey] &&
        this.queryParams[dateStartKey] < ((new Date()).getTime() - (86400000 * 30 * 3))) {
        this.methodNotify('提示', '开始时间不能早于近三个月', 'warning');
        return false;
      }
      if (dateStartKey && this.queryParams[dateStartKey] &&
        this.queryParams[dateStartKey] > (new Date()).getTime()) {
        this.methodNotify('提示', '开始时间不能晚于当前时间', 'warning');
        return false;
      }
      if (this.dateValidateThreeMonth && dateEndKey && this.queryParams[dateEndKey] &&
        this.queryParams[dateEndKey] < ((new Date()).getTime() - (86400000 * 30 * 3))) {
        this.methodNotify('提示', '结束不能早于近三个月', 'warning');
        return false;
      }
      return true;
    },
    methodSearch(method) {
      // request開始
      this.searchLoading = true;
      NProgress.start();
      // 開始驗證
      if (!this.methodDateValidate()) {
        this.searchLoading = false;
        NProgress.done();
        return;
      }
      this.$refs.kbForm.validate().then(
        async () => {
          const params = this.methodInitOutputQueryParams(this.queryParams);
          // 提交請求
          const res = await method(params);
          // request結束
          this.searchLoading = false;
          NProgress.done();
          // 處理response
          if (!res.code) {
            this.$emit('searchEnd', res.data);
          } else {
            // TODO:
          }
        },
        () => {
          this.searchLoading = false;
          NProgress.done();
        },
      );
    },
  },
};
</script>
