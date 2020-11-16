<template>
  <div class="kb-radio-group">
    <label v-if="info.label">
      {{info.label}}
    </label>
    <el-radio-group
      v-model="radioValue"
      @change="methodValueChange()"
      v-bind="$attrs" >
      <span
        v-for="(info, index) in info.group"
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

export default {
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
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      radioValue: null,
    };
  },
  watch: {
    propValue: {
      handler() {
        this.radioValue = this.propValue;
      },
      immediate: true,
    },
  },
  methods: {
    methodValueChange() {
      this.$emit('valueChange', this.radioValue);
    },
  },
};
</script>
<style lang="stylus">
.kb-radio-group
  .el-radio__label
    padding-left 5px
</style>

