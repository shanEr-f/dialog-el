<template>
  <div class="dialog" v-if="visible">
    <div
      :class="['dialog_el']"
      ref="dialogMain"
      @mousedown="evMouseDown"
      :style="{
        left: clientX + 'px',
        top: clientY + 'px',
        width: width + 'px',
        height: height + 'px',
      }"
    >
      <div class="dialog_el_main">
        <!-- 头部 -->
        <div :class="['dialog_header']" v-if="title" @mousedown="evMouseDown">
          {{ title }}
        </div>
        <div class="dialog_fullScreen" @click="evFullScreen" v-if="isIcon">
          <slot name="full" v-if="$slots.full"></slot>
          <dialog-icon :icon="dialogIcon" v-else></dialog-icon>
        </div>
        <!-- 关闭 -->
        <div class="dialog_close" @click="evClose">
          <slot name="close" v-if="$slots.close"></slot>
          <dialog-icon v-else></dialog-icon>
        </div>
        <!-- 内容 -->
        <div :class="['dialog_body']">
          <slot></slot>
        </div>
        <!-- 边框 -->
        <div
          v-for="item in borderDom"
          :key="item.type"
          :class="[item.class, !isZoom ? 'cursor' : '']"
          @mousedown="evBoderDown($event, item.type)"
        ></div>
      </div>
    </div>
    <div class="dialog_mask" v-if="mask"></div>
  </div>
</template>

<script>
import { computed, defineComponent, reactive, ref, toRefs, watch } from "vue";
import dialogIcon from "./componets/dialogIcon.vue";
export default defineComponent({
  components: {
    dialogIcon,
  },
  props: {
    modelValue: {
      type: Boolean,
    },
    title: {
      type: String,
      default: "",
    },
    // 遮罩层
    mask: {
      type: Boolean,
      default: false,
    },
    // 元素默认宽高
    WHStyle: {
      type: Object,
      default: () => {
        return {
          width: window.innerWidth / 2,
          height: window.innerHeight / 2,
        };
      },
    },
    // 是否可以移动
    isMove: {
      type: Boolean,
      default: true,
    },
    // 是否可以拖拽拖放
    isZoom: {
      type: Boolean,
      default: true,
    },
    // 是否展示放大图标
    isIcon: {
      type: Boolean,
      default: true,
    },
  },
  emits: [
    "update:modelValue",
    "evMove",
    "evZoom",
    "evOpen",
    "evClose",
    "evNormal",
    "evFull",
  ],
  setup(props, { slots, emit }) {
    const _data = reactive({
      isDown: false,
      isBoderDown: false,
      // 元素移动为位置
      clientX: 0,
      clientY: 0,
      // 鼠标点击开始位置
      startClientX: 0,
      startClientY: 0,

      // 边框开始点击位置
      boderStartX: 0,
      boderStartY: 0,

      // 元素宽高
      width: 0,
      height: 0,

      // 元素类型
      type: "",

      // 放大后禁止移动
      banMove: false,
    });

    const borderDom = reactive([
      {
        class: "body_bt",
        type: "t",
      },
      {
        class: "body_br",
        type: "r",
      },
      {
        class: "body_bb",
        type: "b",
      },
      {
        class: "body_bl",
        type: "l",
      },
      {
        class: "body_ctl",
        type: "tl",
      },
      {
        class: "body_ctr",
        type: "tr",
      },
      {
        class: "body_cbl",
        type: "bl",
      },
      {
        class: "body_cbr",
        type: "br",
      },
    ]);

    const dialogMain = ref(null);
    const dialogIcon = ref(2);

    // 用于记录缩小的元素的位置
    const sinScreen = reactive({
      clientX: 0,
      clientY: 0,
      width: 0,
      height: 0,
    });

    const bh = computed(() => (props.title ? "40px" : "0%"));
    const WHStyle = computed(() => props.WHStyle);
    const isMove = computed(() => props.isMove);
    const visible = computed(() => props.modelValue);
    const isZoom = computed(() => props.isZoom);

    watch(
      [dialogIcon, visible, _data],
      ([ndialogIcon, nvisible, ndata], [odialogIcon, onvisible]) => {
        // 获取放大之前的元素的数据
        if (ndialogIcon == 2 && nvisible) {
          sinScreen.clientY = _data.clientY;
          sinScreen.clientX = _data.clientX;
          sinScreen.width = _data.width;
          sinScreen.height = _data.height;
        }
      },
      {
        immediate: true,
      }
    );

    // 监听元素打开或者关闭
    watch(visible, (newVal, oldVal) => {
      const _ww = window.innerWidth;
      const _wh = window.innerHeight;
      const _dw = WHStyle.value.width;
      const _dh = WHStyle.value.height;
      _data.clientX = (_ww - _dw) / 2;
      _data.clientY = (_wh - _dh) / 2;
      _data.width = Number(WHStyle.value.width);
      _data.height = Number(WHStyle.value.height);

      if (newVal) {
        emit("evOpen", _data);
      } else {
        emit("evClose", _data);
        _data.banMove = false;
        dialogIcon.value = 2;
      }
    });

    // 监听元素放大和缩小
    watch(dialogIcon, (newVal, oldVal) => {
      newVal == 2 ? emit("evNormal", _data) : emit("evFull", _data);
    });

    // nextTick(() => {
    //   // 获取元素宽高
    //   if (visible.value) {
    //     _data.width = dialogMain.value.clientWidth;
    //     _data.height = dialogMain.value.clientHeight;
    //   }
    // });

    // 移动-鼠标按下
    const evMouseDown = (e) => {
      if (!isMove.value) {
        return false;
      }

      if (_data.banMove) {
        return false;
      }

      _data.isDown = true;
      _data.startClientX = e.clientX;
      _data.startClientY = e.clientY;

      window.addEventListener("mousemove", evMouseMove);
    };

    // 移动-鼠标移动
    const evMouseMove = (e) => {
      if (_data.isDown) {
        const _clientX = e.clientX - _data.startClientX;
        const _clientY = e.clientY - _data.startClientY;

        _data.startClientX = e.clientX;
        _data.startClientY = e.clientY;

        _data.clientX += _clientX;
        _data.clientY += _clientY;

        emit("evMove", _data);
      }
    };

    // 缩放和放大
    const evBoderDown = (e, type) => {
      if (!isZoom.value) {
        return false;
      }

      _data.boderStartX = e.clientX;
      _data.boderStartY = e.clientY;
      _data.isBoderDown = true;
      _data.isDown = false;
      _data.type = type;

      window.addEventListener("mousemove", evBoderMove);
    };

    // 缩放和放大-边框移动
    const evBoderMove = (e) => {
      if (!_data.isBoderDown) {
        return false;
      }

      const _type = _data.type;

      const _clientX = e.clientX - _data.boderStartX;
      const _clientY = e.clientY - _data.boderStartY;

      _data.boderStartX = e.clientX;
      _data.boderStartY = e.clientY;
      _data.isDown = false;

      if (dialogIcon.value == 3) {
        dialogIcon.value = 2;
      }

      switch (_type) {
        case "t":
          _data.height += -_clientY;
          _data.clientY += _clientY;
          break;
        case "r":
          _data.width += _clientX;
          break;
        case "b":
          _data.height += _clientY;
          break;
        case "l":
          _data.width -= _clientX;
          _data.clientX += _clientX;
          break;
        case "tl":
          _data.height += -_clientY;
          _data.clientY += _clientY;
          _data.width -= _clientX;
          _data.clientX += _clientX;
          break;
        case "tr":
          _data.height += -_clientY;
          _data.clientY += _clientY;
          _data.width += _clientX;
          break;
        case "bl":
          _data.height += _clientY;
          _data.width -= _clientX;
          _data.clientX += _clientX;
          break;
        case "br":
          _data.height += _clientY;
          _data.width += _clientX;
          // _data.clientX += _clientX;
          break;
        default:
          break;
      }

      emit("evZoom", _data);
    };

    // 鼠标抬起
    window.addEventListener("mouseup", () => {
      _data.isDown = false;
      _data.isBoderDown = false;
      window.removeEventListener("mousemove", evBoderMove);
      window.removeEventListener("mousemove", evMouseMove);
    });

    // 关闭弹窗
    const evClose = (e) => {
      emit("update:modelValue", false);
    };

    // 全屏
    const evFullScreen = (e) => {
      dialogIcon.value = dialogIcon.value == 3 ? 2 : 3;

      if (dialogIcon.value == 3) {
        _data.width = window.innerWidth;
        _data.height = window.innerHeight;
        _data.clientX = 0;
        _data.clientY = 0;
      } else {
        _data.width = sinScreen.width;
        _data.height = sinScreen.height;
        _data.clientX = sinScreen.clientX;
        _data.clientY = sinScreen.clientY;
      }
    };

    document.onselectstart = function () {
      if (isMove.value) {
        return false;
      }
    };

    return {
      // methods
      evMouseDown,
      evMouseMove,
      evClose,
      evBoderDown,
      evBoderMove,
      evFullScreen,

      // computed
      bh,
      visible,
      dialogIcon,

      // data
      ...toRefs(_data),
      sinScreen,
      // 边框
      borderDom,
      // ref
      dialogMain,
    };
  },
});
</script>

<style scoped>
.dialog_mask {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background: #00000080;
  z-index: 998;
}

.dialog_el {
  position: fixed;
  background: #fff;
  min-width: 177px;
  min-height: 189px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  transition: top left 0.5s;
  z-index: 999;
}

.dialog_el_auto {
  right: 0;
  margin: 10% auto 0;
}

.dialog_el_main {
  width: 100%;
  height: 100%;
  position: relative;
  /* user-select: none; */
}

.dialog_header {
  max-width: 50%;
  height: 40px;
  display: flex;
  align-items: center;
  padding: 0 10px;
  box-sizing: border-box;
  overflow: hidden;
  /* user-select: none; */
  white-space: nowrap;
  text-overflow: ellipsis;
}

.dialog_body {
  width: 100%;
  height: calc(100% - v-bind(bh));
  /* background: yellowgreen; */
  padding: 10px;
  box-sizing: border-box;
  /* user-select: none; */
}

.dialog_fullScreen {
  width: 20px;
  height: 20px;
  position: absolute;
  top: 16px;
  right: 50px;
  cursor: pointer;
}

.dialog_close {
  width: 20px;
  height: 20px;
  position: absolute;
  top: 16px;
  right: 16px;
  cursor: pointer;
}

/* 边框 */
.body_bt,
.body_bb,
.body_br,
.body_bl {
  position: absolute;
  /* background: rgba(0, 0, 0, 1); */
  z-index: 999;
}

.body_ctl,
.body_ctr,
.body_cbl,
.body_cbr {
  width: 8px;
  height: 8px;
  /* background: red; */
  z-index: 999;
  position: absolute;
}

.body_bt,
.body_bb {
  width: 100%;
  height: 2px;
  left: 0;
}

.body_br,
.body_bl {
  width: 2px;
  top: 0;
  bottom: 0;
}

.body_bt {
  top: 0;
  cursor: n-resize;
}

.body_bb {
  bottom: 0;
  cursor: s-resize;
}

.body_bl {
  left: 0;
  cursor: w-resize;
}

.body_br {
  right: 0;
  cursor: e-resize;
}

.body_ctl,
.body_cbl {
  left: 0;
}

.body_ctr,
.body_cbr {
  right: 0;
}

.body_ctl,
.body_ctr {
  top: 0;
}

.body_cbl,
.body_cbr {
  bottom: 0;
}

.body_ctl {
  cursor: nw-resize;
}

.body_ctr {
  cursor: ne-resize;
}

.body_cbr {
  cursor: se-resize;
}

.body_cbl {
  cursor: sw-resize;
}

.cursor {
  cursor: auto !important;
}
</style>

