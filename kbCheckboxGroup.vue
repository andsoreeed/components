<template>
  <kb-col class="kb-checkbox-group">
    <label v-if="info.label">
      {{info.label}}
    </label>
    <el-checkbox-group
      v-model="checkList"
      @change="methodValueChange()"
      v-bind="$attrs" >
      <div
        v-for="(info, index) in info.group"
        :key="index" >
        <el-checkbox
          :label="info.value"
          size="small"
          @change="handleChange($event, info)"
          :indeterminate="isArray(checkList)
            &&!checkList.includes(info.value)
            && (
              (!!info.popover && info.popover.part)
              || (!!info.popover && info.popover.value.length > 0)
            )" >
          {{info.label}}
        </el-checkbox>
        <template v-if="info.popover">
          <el-popover
            :key="`checkbox-popover-${index}`"
            :title="info.popover.title || ''"
            :width="info.popover.width || 550"
            placement="right-start"
            trigger="manual"
            @show="handlePopoverShow(true, index, info)"
            @hide="handlePopoverShow(false, index, info)"
            v-model="popoverShow[index]" >
            <kb-view-popover
              :data="info.popover"
              ref="kbViewPopover" />
            <span
              style="font-size: 14px; cursor: pointer;"
              @click="triggerPopover($event, index, info)"
              slot="reference" >
              {{info.popover.text}}
            </span>
          </el-popover>
        </template>
      </div>
    </el-checkbox-group>
  </kb-col>
</template>
<script>
import { isFunction, keys, isArray } from 'lodash';
import kbViewPopover from '@/components/kbViewPopover.vue';
import kbCol from './kbCol.vue';

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
  components: {
    kbCol,
    kbViewPopover,
  },
  data() {
    return {
      checkList: [],
      popoverShow: this.info.group.map(() => false),
    };
  },
  watch: {
    propValue: {
      handler() {
        this.checkList = this.propValue;
      },
      immediate: true,
    },
  },
  methods: {
    isArray,
    methodValueChange() {
      this.$emit('valueChange', this.checkList);
    },
    handleChange(v, info) {
      const { popover } = info;
      if (popover) {
        const { initMethod } = popover;
        if (isFunction(initMethod)) initMethod(v, info);
      }
    },
    handlePopoverShow(show, index, info) {
      this.$refs.kbViewPopover[index].changeShow(show);
      const { popover: { method } } = info;
      if (isFunction(method)) method(show, info);
    },
    handleEvent(event, byHand = false) {
      if (byHand || !/el-popover__reference/.test(event.target.classList.value)) {
        this.popoverShow.splice(this.popoverShow.findIndex(v => v), 1, false);
        document.body.removeEventListener('click', this.handleEvent);
      }
    },
    async triggerPopover(event, index, info) {
      const { popover: { preData } } = info;
      if (this.popoverShow.find(v => v)) {
        this.handleEvent(event, true);
      }
      if (!this.popoverShow[index]) document.body.addEventListener('click', this.handleEvent);

      if (isFunction(preData)) await preData(info.popover);
      this.popoverShow.splice(index, 1, !this.popoverShow[index]);
    },
  },
};
</script>
