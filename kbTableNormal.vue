<template>
  <div class="kb-table-normal" v-bind:style="{ 'white-space': item.wrap ? 'wrap' : 'nowrap' }">
    <!-- 一般列，需過方法格式化 -->
    <span v-if="item.method">
      <template
        v-if="imgUrlValidator && imgUrlValidator.test(item.method(judgeMethodParam()))">
        <img :src="item.method(judgeMethodParam())" />
      </template>
      <template v-else>{{item.method(judgeMethodParam())}}</template>
    </span>
    <!-- 一般列，圖片 -->
    <span v-else-if="imgUrlValidator && imgUrlValidator.test(scope.row[item.dataKey])">
      <img :src="scope.row[item.dataKey]" />
    </span>
    <!-- 一般列，文字 -->
    <span v-else>{{scope.row[item.dataKey]}}</span>
  </div>
</template>
<script>
import { has } from 'lodash';

export default {
  props: ['item', 'scope'],
  // data() {
  //   return {
  //     imgUrlValidator: new RegExp(`^(${process.env.VUE_APP_IMGURL})/\\w+(/0)$`),
  //   };
  // },
  computed: {
    imgUrlValidator() {
      if (typeof process !== 'undefined' && process.env && process.env.VUE_APP_IMGURL) {
        return new RegExp(`^(${process.env.VUE_APP_IMGURL})/(\\w|\\.)+(/0)$`);
      }
      return this.item.imgUrlValidator || null;
    },
  },
  methods: {
    judgeMethodParam() {
      const { scope: { row }, item: { dataKey } } = this;
      if (has(row, dataKey)) return row[dataKey];
      return dataKey === '' ? row : undefined;
    },
  },
};
</script>
