## dialog-el

> 一个vue3.x的可以拖拽移动， 拖拽缩小和放大的dialog组件

### 安装

```
npm i dialog-el
```



### 演示
https://wangdageeee.gitee.io/componentel/



### 使用

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import './index.css'
import dialogEl from "dialog-el"

createApp(App).use(dialogEl).mount('#app');

```

```vue
<!-- Vue -->
<template>
  <button @click="evClick">打开dialog</button>
  <button @click="evMask">打开遮罩层</button>
  <dialog-el
    title="这是一个标题"
    v-model="isShow"
    :mask="isMask"
    @evOpen="open"
  >
    <div class="box">
      鼠标移动至盒子边缘（鼠标样式发生改变）,
      按住鼠标左键即可拖拽缩放和放大；鼠标移动到盒子内部按住鼠标左键即可移动；鼠标左键抬起事件取消
    </div>
  </dialog-el>
</template>

<script>
import { reactive, ref } from "vue";
export default {
  setup() {
    const isShow = ref(false);
    const isMask = ref(false);

    const evClick = () => {
      isShow.value = true;
    };

    const evMask = () => {
      isMask.value = !isMask.value;
    };

    const open = (data) => {
      console.log("我打开了", data);
    };

    return {
      isShow,
      isMask,

      evClick,
      evMask,
      open,
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
```



### 属性

| 属性    | 说明                           | 类型    | 默认值                                                       |
| ------- | ------------------------------ | ------- | ------------------------------------------------------------ |
| v-model | 是否打开dialogEl               | Boolean | --                                                           |
| title   | 标题                           | String  | --                                                           |
| mask    | 是否展示dialogEl的遮罩层       | Boolean | false                                                        |
| WHStyle | dialogEl的默认宽高(无需填写px) | Object  | { width: window.innerWidth / 2,height: window.innerHeight / 2} |
| isMove  | dialogEl是否可以移动           | Boolean | true                                                         |
| isZoom  | dialogEl是否可以拖拽缩放       | Boolean | true                                                         |
| isIcon  | dialogEl是否展示放大图标       | Boolean | true                                                         |



### 插槽

| 插槽名 | 说明               | 默认             |
| ------ | ------------------ | ---------------- |
| --     | dialogEl的内容     | --               |
| full   | dialogEl的放大图标 | dialogEl自带图标 |
| close  | dialogEl的关闭图标 | dialogEl自带图标 |



### 方法

| 方法     | 说明                             | 回调参数 |
| -------- | -------------------------------- | -------- |
| evOpen   | dialogEl打开的回调               | data     |
| evClose  | dialogEl关闭的回调               | data     |
| evMove   | dialogEl移动的回调               | data     |
| evZoom   | dialogEl拖拽缩放的回调           | data     |
| evFull   | dialogEl最大化的回调             | data     |
| evNormal | dialogEl恢复为正常窗口时候的回调 | data     |



### Tip

> 鼠标移动至盒子边缘（鼠标样式发生改变）, 按住鼠标左键即可拖拽缩放和放大；鼠标移动到盒子内部按住鼠标左键即可移动；鼠标左键抬起事件取消

> 由于选中文本会导致元素移动和缩放出现问题，所以只能在dialogEl禁止移动的时候才可以选中文件

