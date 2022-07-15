<template>
  <div class="container" ref="wrap">
    <slot></slot>
    <div
      class="popover"
      v-show="show"
      ref="content"
      :style="popoverStyle"
      oncontextmenu="return false"
      :class="popoverClass"
    >
      <slot name="content">{{ content }}</slot>
    </div>
  </div>
</template>

<script>
const eventMap = {
  contextmenu: "contextmenu",
  click: "click",
  hover: "mouseover",
};
export default {
  components: {},
  props: {
    trigger: {
      type: String,
      default: "contextmenu",
      validator: (value) => ["contextmenu", "click", "hover", "manual"].includes(value),
    },
    popoverClass: String,
    content: String,
    placement: {
      type: String,
      default: "bottom",
      validator: (value) =>
        [
          "top",
          "top-start",
          "top-end",
          "bottom",
          "bottom-start",
          "bottom-end",
          "left",
          "left-start",
          "left-end",
          "right",
          "right-start",
          "right-end",
        ].includes(value),
    },
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      popoverStyle: {},
      show: false,
      w: 0,
      h: 0,
    };
  },
  watch: {
    value: {
      immediate: true,
      handler (visible) {
        if (this.trigger === 'contextmenu') return
        if(visible) {
          this.$nextTick(() => this.handleEvent())
        } else {
          this.show = false
        }
      }
    }
  },
  computed: {
    eventName() {
      return eventMap[this.trigger];
    },
    hiddenEvent() {
      if (this.trigger === "hover") {
        return "mousemove";
      }
      return "click";
    },
  },
  mounted() {
    document.body.appendChild(this.$refs.content);
    // 用于监听点击/鼠标移出悬浮框外，以隐藏悬浮框
    this.trigger !== 'manual' && document.body.addEventListener(this.hiddenEvent, this.handleHiddenEvent);
    // 给目标元素绑定trigger事件
    this.trigger !== 'manual' && this.$refs.wrap.firstChild.addEventListener(this.eventName, this.handleEvent)
    // 浏览器窗口宽度：
    this.w =
      window.innerWidth ||
      document.documentElement.clientWidth ||
      document.body.clientWidth;

    // 浏览器窗口高度：
    this.h =
      window.innerHeight ||
      document.documentElement.clientHeight ||
      document.body.clientHeight;

    if (this.trigger !== "contextmenu") {
      this.watchScroll();
    }
  },
  destroyed() {
    this.trigger !== 'manual' && document.body.removeEventListener(this.hiddenEvent, this.handleHiddenEvent);
    this.trigger !== 'manual' && this.$refs.wrap.firstChild.removeEventListener(this.eventName, this.handleEvent)
  },
  methods: {
    // trigger事件
    async handleEvent(event) {
      this.show = true;
      if (this.trigger === "contextmenu") {
        this.popoverStyle = await this.getContextmenuStyle(event);
      } else {
        this.popoverStyle = await this.getHoverOrClickStyle();
      }
    },
    // 获取触发元素的size
    getInnerSize() {
      let wrapDom = this.$refs.wrap;
      let innerDom = wrapDom.firstChild;
      return innerDom.getBoundingClientRect();
    },
    // 隐藏悬浮框的事件
    handleHiddenEvent(event) {
      let target = event.target;
      if (
        ["click", "hover"].includes(this.trigger) &&
        !this.$refs.content.contains(target) &&
        !this.$refs.wrap.firstChild.contains(target)
      ) {
        this.show = false;
        this.$emit('input', false)
      } else if (
        ["contextmenu"].includes(this.trigger) &&
        !this.$refs.content.contains(target)
      ) {
        this.show = false;
      }
    },
    async getContextmenuStyle({ clientX, clientY }) {
      let contentDom = this.$refs.content;
      let width = 0;
      let height = 0;
      await this.$nextTick();
      width = contentDom.offsetWidth;
      height = contentDom.offsetHeight;
      let style = {};
      if (clientX + width > this.w) {
        style.left = Number(clientX - width) + "px";
      } else {
        style.left = clientX + "px";
      }
      if (clientY + height > this.h) {
        style.bottom = 0 + "px";
      } else {
        style.top = clientY + "px";
      }
      return style;
    },
    async getHoverOrClickStyle() {
      let { height, top, left, width } = this.getInnerSize();
      let contentDom = this.$refs.content;
      let contentHeight = 0;
      let contentWidth = 0;
      // let contentWidth = 0
      await this.$nextTick();
      contentHeight = contentDom.offsetHeight
      contentWidth = contentDom.offsetWidth
      let style = {};
      style.left = this.getLeft({ height, top, left, width, contentHeight, contentWidth })
      // if (top + contentHeight + height > this.h) {
      //   style.bottom = top + "px";
      // } else {
      //   style.top = top + height + "px";
      // }
      // style.position = "absolute";
      style.top = this.getTop({ height, top, left, width, contentHeight, contentWidth })
      return style;
    },
    getLeft ({ height, top, left, width, contentHeight, contentWidth }) {
      let leftMap = {
        'top': Number(left - Math.abs(contentWidth - width) / 2) + 'px',
        'top-start': left + 'px',
        'top-end': Number(left + width - contentWidth) + 'px',
        'bottom': Number(left - Math.abs(contentWidth - width) / 2) + 'px',
        'bottom-start': left + 'px',
        'bottom-end': Number(left + width - contentWidth) + 'px',
        "right": Number(left + width) + 'px',
        "right-start": Number(left + width) + 'px',
        "right-end": Number(left + width) + 'px',
        "left": Number(left - contentWidth) + 'px',
        "left-start": Number(left - contentWidth) + 'px',
        "left-end": Number(left - contentWidth) + 'px',
      }
      return leftMap[this.placement] || 'initial'
    },
    getTop ({ height, top, left, width, contentHeight, contentWidth }) {
      let topMap = {
        'bottom': Number(top + height) + 'px',
        'bottom-end': Number(top + height) + 'px',
        'bottom-start': Number(top + height) + 'px',
        "right": Number(top - Math.abs(height - contentHeight) /2 ) + 'px',
        "right-start": top + 'px',
        "right-end": Number(top + height - contentHeight) + 'px',
        'top': Number(top - contentHeight) + 'px',
        'top-start': Number(top - contentHeight) + 'px',
        'top-end': Number(top - contentHeight) + 'px',
        "left":  Number(top - Math.abs(height - contentHeight) /2 ) + 'px',
        "left-start": top + 'px',
        "left-end": Number(top + height - contentHeight) + 'px',
      }
      return topMap[this.placement] || 'initial'
    },
    // 递归查找离目标元素最近的内容溢出的dom
    getScrollParentDom(dom) {
      let parent = dom.parentNode || dom.parentElement;
      if (!parent) {
        return dom;
      }
      if (
        ["scroll", "auto"].includes(parent.style.overflow) ||
        ["scroll", "auto"].includes(parent.style.overflowY) ||
        ["scroll", "auto"].includes(parent.style.overflowX)
      ) {
        return parent;
      } else {
        return this.getScrollParentDom(parent);
      }
    },
    watchScroll() {
      let scrollParent = this.getScrollParentDom(this.$refs.wrap.firstChild);
      scrollParent.addEventListener("scroll", this.updateClickOrHoverPosition);
    },
    async updateClickOrHoverPosition() {
      this.popoverStyle = await this.getHoverOrClickStyle();
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
}
.popover {
  position: fixed;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  background-color: #fff;
  width: 200px;
  border-radius: 4px;
  border: 1px solid #ebeef5;
  padding: 12px;
  z-index: 9999000;
  color: #606266;
  /* height: 500px; */
}
</style>
