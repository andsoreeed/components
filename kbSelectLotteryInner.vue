<template>
  <el-select
    v-model="value"
    class="kb-select-lottery-inner"
    :multiple="info.multiple"
    :disabled="disabled && info.disabled"
    @change="selectChange"
    v-bind="$attrs"
    popper-class="lottery-popper"
    :clearable="true"
    :filterable="true"
    size="small">
    <el-option-group
      v-for="group in options"
      :key="`lottery-group-${group.name}`"
      :label="group.name" >
      <el-option
        v-for="item in group.group"
        :key="`lottery-${item.name}`"
        :label="item.name"
        :value="item.cid" />
    </el-option-group>
  </el-select>
</template>
<script>
import { mapGetters } from 'vuex';
import { isArray, isObject } from 'lodash';
import { getLottery } from '@/api/global';

const invalid = [null, undefined];

export default {
  name: 'kbSelectLotteryInner',
  inheritAttrs: false,
  model: {
    prop: 'propValue',
    event: 'valueChange',
  },
  props: {
    propValue: {
      required: true,
    },
    info: {
      required: true,
      default: () => {},
    },
  },
  data() {
    return {
      value: this.propValue,
      options: [],
      disabled: true,
    };
  },
  async mounted() {
    await this.getLotteryOption(this.info.getParams);
    if (this.value) this.selectChange(this.value);
  },
  watch: {
    value(val) { this.$emit('valueChange', val); },
    propValue: {
      handler(val) {
        if (isArray(val) && isArray(this.value) && val.join('') === this.value.join('')) return;
        else if (!isArray(val) && !isArray(this.value) && val === this.value) return;
        this.value = val;
        this.selectChange(val);
      },
      immediate: true,
    },
  },
  methods: {
    selectChange(value) {
      const { info: { method, multiple = false } } = this;
      if (method) {
        if (multiple && value.length !== 0) method(value);
        else if (!multiple && !invalid.includes(value)) method(value);
      }
    },
    async getLotteryOption(params) {
      const res = await getLottery(isObject(params) ? params : {});
      if (res.code !== 0) {
        this.fNotify('提示', res.msg, 'error');
        return;
      }
      const { data } = res;
      this.options = this.outputLotteryOption(data);
      this.disabled = false;
    },
    outputLotteryOption(data) {
      return data.reduce((acc, cur) => {
        const specified = acc.find(({ name }) => name === cur.lotteryTypeName);
        if (specified) specified.group.push(cur);
        else acc.push({ name: cur.lotteryTypeName, group: [cur] });
        return acc;
      }, []);
    },
  },
};
</script>
<style lang="stylus">
.lottery-popper
  .el-select-group__wrap
    &:not(:last-of-type)
      padding-bottom 5px
      &::after
        bottom 0px
    .el-select-group__title
      line-height 20px
    .el-select-dropdown__item
      height 20px
      line-height 20px
</style>
