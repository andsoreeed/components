<template>
  <div class="kb-tabs-group">
    <el-tabs
      v-model="activeName"
      :type="info.tabsType"
      :tab-position="info.tabPosition"
      @tab-click="methodSelectTabChange($event, info.method)" >
      <el-tab-pane
        v-for="(item, index) in info.group"
        :key="index"
        :label="item.label"
        :ref="`tabPane${item.tabsName}`"
        :name="item.tabsName" >
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
export default {
  model: {
    prop: 'propValue',
    event: 'valueChange',
  },
  props: {
    // v-model綁定值
    propValue: {
      required: true,
      default: '',
    },
    info: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      activeName: this.propValue,
    };
  },
  watch: {
    activeName(val) {
      this.$emit('valueChange', val);
    },
    propValue(val) {
      if (val !== this.activeName) {
        this.activeName = String(this.propValue);
        this.$nextTick(() => {
          setTimeout(() => {
            this.methodSelectTabChange(this.$refs[`tabPane${this.activeName}`][0], this.info.method);
          });
        });
      }
    },
  },
  methods: {
    async methodSelectTabChange($event, method) {
      if (method) {
        const res = await method($event);
        if (res) {
          //
        }
      }
    },
  },
};
</script>
