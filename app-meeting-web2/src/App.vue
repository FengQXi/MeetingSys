<template>
  <div id="app">

    <router-view/>
  </div>
</template>

<style>
::-webkit-scrollbar {
  width: 6px;
  height: 8px;
  background-color: #ebeef5;
}
::-webkit-scrollbar-thumb {
  box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
  background-color: #ccc;
}
::-webkit-scrollbar-track{
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 3px;
  background: rgba(255, 255, 255, 1);
}
/* 弹出框弹出后body增加padding-right？？？解决方法 */
body {
  padding-right: 0 !important;
}
/* 组件适配—Message 消息提示 */
@media screen and (max-width: 600px) {
    .el-message {
      min-width: 300px !important;
    }
}
@media screen and (max-width: 500px) {
  .el-message-box {
    width: 300px !important;
  }
}
</style>
