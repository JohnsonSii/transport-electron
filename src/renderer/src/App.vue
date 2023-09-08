<template>
  <div class="content">
    <div class="menu">
      <a-menu v-model:selectedKeys="state" class="boom" theme="dark" @select="handleSelect">
        <div class="header center">Transport</div>
        <a-menu-item key="1">
          <div class="menu-item">
            <img src="./assets/img/send.svg" />
            <span>上传</span>
          </div>
        </a-menu-item>
        <a-menu-item key="2">
          <div class="menu-item">
            <img src="./assets/img/download.svg" />
            <span>下载</span>
          </div>
        </a-menu-item>
        <a-menu-item key="3">
          <div class="menu-item">
            <img src="./assets/img/setting.svg" />
            <span>设置</span>
          </div>
        </a-menu-item>
      </a-menu>
    </div>
    <div class="component --1">
      <Uploader v-if="currentKey === 1" />
      <Setting v-if="currentKey === 3" />
      <Download v-if="currentKey === 2" />
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import Uploader from './components/Uploader.vue'
import Setting from './components/Setting.vue'
import Download from './components/Download.vue'

const state = reactive(['1'])
const currentKey = ref(1)

const handleSelect = (e) => {
  currentKey.value = Number(e.key)
  state[0] = e.key
  console.log(state)
}
</script>

<style lang="scss" scoped>
@import './assets/css/global.scss';
.content {
  height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: row;
}
.menu {
  width: 230px;
  height: 100%;
}
.header {
  height: 100px;
  font-size: 28px;
  color: white;
  font-weight: 600;
  user-select: none;
}
.menu-item {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  margin-left: 5px;
}
.menu-item span {
  margin-left: 15px;
}
</style>
