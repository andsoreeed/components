<template>
  <div class="kb-table-index" style="white-space: nowrap">
    <span v-if="item.methodIndex">{{item.methodIndex(scope.$index)}}</span>
    <span v-else>{{scope.$index + 1}}</span>
  </div>
</template>
<script>
export default {
  props: ['item', 'scope'],
};
</script>

