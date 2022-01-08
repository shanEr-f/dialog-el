<template>
  <button @click="evClick">打开dialog</button>

  <dialog-el
    title="这是一个标题"
    v-model="isShow"
    :mask="isMask"
    :isMove="isMove"
    :isZoom="isZoom"
    @evOpen="open"
    @evFull="full"
    @evNormal="normal"
    @evZoom="move"
  >
    <div class="box">
      <button @click="evMask">打开/关闭遮罩层</button>
    </div>
  </dialog-el>
</template>

<script>
import { reactive, ref } from "vue";
import dialogEl from "./DialogEl/index.vue";
export default {
  components: {
    dialogEl,
  },
  setup() {
    const isShow = ref(false);
    const isMask = ref(false);
    const isMove = ref(true);
    const isZoom = ref(true);

    const evClick = () => {
      console.log(isZoom);
      isShow.value = true;
    };

    const evMask = () => {
      isMask.value = !isMask.value;
    };

    const open = (data) => {
      console.log("我打开了", data);
    };

    const whStyle = reactive({
      width: window.innerWidth,
      height: window.innerHeight,
    });

    const full = (data) => {
      isMove.value = false;
      isZoom.value = false;
    };

    const normal = (data) => {
      isMove.value = true;
      isZoom.value = true;
    };

    const move = (data) => {
      isMove.value = isMove.value ? isMove.value : !isMove.value;
    };

    return {
      isShow,
      isMask,
      isMove,
      isZoom,

      evClick,
      evMask,
      open,
      whStyle,
      full,
      normal,
      move,
    };
  },
};
</script>

<style scoped>
.box {
  width: 100%;
  height: 100%;
  background: palegoldenrod;
  overflow-y: auto;
}
</style>