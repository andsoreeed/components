<template>
  <div class="kb-table-radio-group">
    <label v-if="item.label">
      {{item.label}}
    </label>
    <el-radio-group
      v-model="formData.data[scope.$index][item.dataKey]"
      @change="methodValueChange()"
    >
      <span
        v-for="(info, index) in item.group"
        :key="`radio-${index}`"
        style="margin-right: 10px" >
        <template v-if="info.type === 'radio'">
          <el-radio
            :label="info.value" >
            {{info.label}}
          </el-radio>
        </template>
        <template v-if="info.type === 'button'">
          <el-radio-button
            :label="info.value" >
            {{info.label}}
          </el-radio-button>
        </template>
      </span>
    </el-radio-group>
  </div>
</template>
<script>

import { isFunction } from 'lodash';

export default {
  props: ['item', 'scope'],
  inject: {
    formData: {
      default: () => null,
    },
  },
  methods: {
    methodValueChange() {
      if (isFunction(this.item.valueChangeCb)) {
        this.item.valueChangeCb(this.formData.data[this.scope.$index][this.item.dataKey]);
      }
    },
  },
};
</script>

<style lang="stylus">
.kb-table-radio-group
  width 100px
  .el-radio__label
    padding-left 5px
    font-size 12px
    font-weight 0
</style>

