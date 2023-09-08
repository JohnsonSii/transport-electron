<template>
  <div class="content3 center">
    <div class="title">
      所有人：
      <a-select ref="select" v-model:value="value">
        <a-select-option v-for="item in selectOptions" :key="item" :value="item.name">
          {{ item.name }}</a-select-option
        >
      </a-select>
    </div>
    <div class="rowcenter">
      <a-input
        v-model:value="fileNameValue"
        style="width: 200px; height: 32px; margin-right: 20px; padding-left: 10px"
        placeholder="请输入文件名"
      ></a-input>
      <a-button type="primary" style="width: 60px" @click="handleDownload">下载</a-button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
const { ipcRenderer } = require('electron')
const fs = require('fs')
import { message } from 'ant-design-vue'

const fileNameValue = ref('')
const downloadPath = ref('')
const filePath = './config.json'
const value = ref('')
const globalConfig = ref()
const selectOptions = ref([])
const currentDownloadUrl = ref('')

if (fs.existsSync(filePath)) {
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    try {
      const config = JSON.parse(data)
      console.log(config)
      downloadPath.value = config.downloadPath
      globalConfig.value = config
      selectOptions.value = config.api
      value.value = selectOptions.value[0].name
      currentDownloadUrl.value = selectOptions.value[0].download
    } catch (e) {
      console.error('Error parsing JSON', e)
    }
  })
}
const handleDownload = () => {
  console.log(downloadPath.value)
  if (!fileNameValue.value) {
    message.error('文件名不能为空')
    return
  }

  ipcRenderer
    .invoke('save-dialog', {
      defaultPath: downloadPath.value,
      defaultName: fileNameValue.value,
      url: `${currentDownloadUrl.value}?fileName=${fileNameValue.value}`
    })
    .then((response) => {
      if (response.success) {
        message.success('文件保存成功')
      } else {
        message.error('文件保存失败')
      }
    })
    .catch(() => {
      message.error('文件保存失败')
    })
}

watch(value, (_val) => {
  selectOptions.value.forEach((item) => {
    if (item.name === _val) {
      currentDownloadUrl.value = item.download
    }
  })
})
</script>

<style lang="scss" scoped>
@import '../assets/css/global.scss';
.content3 {
  width: 100%;
  height: 100vh;
}
.title {
  position: absolute;
  top: 75px;
  font-size: 14px;
  color: #555555;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  font-weight: 600;
  z-index: 50;
}
</style>
