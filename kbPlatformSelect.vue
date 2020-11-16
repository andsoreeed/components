<template>
  <div class="kb-platform-select">
    <kb-select
      v-model="value"
      :info="setting"></kb-select>
  </div>
</template>

<script>
import { get, isArray } from 'lodash';
import { mapState, mapGetters } from 'vuex';
import store from '@/store';
import kbSelect from './kbSelect.vue';

export default {
  components: {
    kbSelect,
  },
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
    info: Object,
  },
  data() {
    return {
      value: this.propValue,
      setting: {
        label: this.info.halltype ? '厅别' : '游戏平台',
        dataKey: this.info.halltype ? 'gameHalltype' : 'gamePlatformName',
        options: [],
        multiple: !this.info.halltype,
        placeholder: '请选择',
      },
    };
  },
  computed: {
    ...mapState({ halltypePage: state => state.global.halltypeplatformOptionHalltypePage }),
    ...mapGetters({
      optionObj: 'global/halltypeplatformOptionObj',
      // halltypeOption: 'global/halltypeOption',
    }),
  },
  watch: {
    value(val) {
      if (this.info.halltype && this.info.page) {
        this.$store.commit(
          'global/setHalltypeplatformOptionHalltypePage',
          { [this.info.page]: val || 0 },
        );
      }
      this.$emit('valueChange', val);
    },
    halltypePage: {
      handler(val) {
        if (this.info.halltype) {
          const allOptions = [{ id: 0, value: '全部' }].concat(store.getters['global/halltypeOption']);
          // const allOptions = [{ id: 0, value: '全部' }].concat(this.halltypeOption);
          this.setting.options = allOptions;
        } else {
          const typeId = val[this.info.page] || 0;
          this.setting.options = this.optionObj[typeId];
        }
      },
      immediate: true,
    },
    propValue: {
      handler(val) {
        if (!isArray(val) && val && isArray(this.setting.options) &&
          !this.setting.options.find(option => option.id === val)) {
          this.$emit('valueChange', null);
        } else if (val !== this.value) {
          this.value = this.propValue;
        }
      },
      immediate: true,
    },
  },
  created() {
  },
};
</script>
