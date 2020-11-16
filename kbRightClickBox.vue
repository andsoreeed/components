<template>
  <div
    :id="boxId"
    class="kb-right-click-box"
    v-show="boxShow" >
    <slot />
  </div>
</template>
<script>
import uuid from 'uuidv4';

export default {
  name: 'kbRightClickBox',
  data() {
    return {
      boxId: uuid(),
      boxShow: false,
    };
  },
  methods: {
    // 對外開放方法，打開右鍵小彈窗
    open(event, cb) {
      if (!this.boxShow) {
        cb();
        this.openBox(event);
      }
    },
    // 對外開放方法，手動關閉彈窗
    close() {
      this.boxShow = false;
      document.body.removeEventListener('mousedown', this.checkNextClickDom);
    },
    // 浏览器的可见高度
    getClientHeight() {
      let clientHeight = 0;
      if (document.body.clientHeight && document.documentElement.clientHeight) {
        clientHeight = (document.body.clientHeight < document.documentElement.clientHeight)
          ? document.body.clientHeight : document.documentElement.clientHeight;
      } else {
        clientHeight = (document.body.clientHeight > document.documentElement.clientHeight)
          ? document.body.clientHeight : document.documentElement.clientHeight;
      }
      return clientHeight;
    },
    // 浏览器的可见宽度
    getClientWidth() {
      let clientWidth = 0;
      if (document.body.clientWidth && document.documentElement.clientWidth) {
        clientWidth = (document.body.clientWidth < document.documentElement.clientWidth)
          ? document.body.clientWidth : document.documentElement.clientWidth;
      } else {
        clientWidth = (document.body.clientWidth > document.documentElement.clientWidth)
          ? document.body.clientWidth : document.documentElement.clientWidth;
      }
      return clientWidth;
    },
    openBox(event) {
      if (!this.boxShow) {
        this.boxShow = true;
        // nextTick才抓的到box的宽高
        this.$nextTick(() => {
          const box = document.getElementById(this.boxId);
          const boxInfo = box.getBoundingClientRect();
          // box宽高
          const boxH = boxInfo.height;
          const boxW = boxInfo.width;
          // 客户设备宽高
          const clientH = this.getClientHeight();
          const clientW = this.getClientWidth();
          // 右键点击时x与y轴
          const clickH = event.clientY;
          const clickW = event.clientX;
          let setH = clickH;
          let setW = clickW;
          if (clickH + boxH > clientH) setH = clientH - boxH - 10;
          if (clickW + boxW > clientW) setW = clientW - boxW - 10;
          box.style.top = `${setH}px`;
          box.style.left = `${setW}px`;
          document.body.addEventListener('mousedown', this.checkNextClickDom);
        });
      }
    },
    checkNextClickDom(e) {
      const findTarget = (node, target) => {
        if (node) {
          if (node.className && node.className.includes(target)) return node;
          return findTarget(node.parentNode, target);
        }
        return false;
      };
      if (findTarget(e.target, 'kb-right-click-box') === false) {
        this.boxShow = false;
        document.body.removeEventListener('mousedown', this.checkNextClickDom);
      }
    },
  },
};
</script>
<style lang="stylus">
.kb-right-click-box
  z-index 999999
  position fixed
  border-radius 0.3rem
  background-color #2a3c55
  padding 5px 0
  box-shadow 0 0 5px rgba(0,0,0,.2)
  & > div
    width auto
    padding 0 1rem
    color white
    cursor pointer
    text-align center
    &:hover
      background-color #44a9c5
</style>
