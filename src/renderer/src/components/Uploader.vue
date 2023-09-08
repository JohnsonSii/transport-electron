<template>
  <div class="content center">
    <div class="card center" @drop="handleUpload($event)" @dragover="handleDragOver($event)">
      <div class="title">
        所有人：
        <a-select ref="select" v-model:value="value">
          <a-select-option v-for="item in selectOptions" :key="item" :value="item.name">
            {{ item.name }}</a-select-option
          >
        </a-select>
      </div>
      <div class="circle1 ping"></div>
      <div class="circle2 ping"></div>
      <div class="circle3 pulsate-fwd"></div>
      <div class="circle4"></div>
      <div
        v-for="i in 100"
        :key="i"
        :style="{
          height: i * 150 + 'px',
          width: i * 150 + 'px',
          bottom: 190 - (i * 150) / 2 + 'px'
        }"
        class="circle"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, createVNode } from 'vue'
import axios from 'axios'
import { message, Modal } from 'ant-design-vue'
import { ExclamationCircleOutlined } from '@ant-design/icons-vue'

const fs = require('fs')
const os = require('os')
const path = require('path')
const value = ref('')

const filePath = './config.json'

const downloadsFolder = path.join(os.homedir(), 'Downloads')
const downloadValue = ref(downloadsFolder)

const globalConfig = ref()
const selectOptions = ref([])
const currentUploadUrl = ref('')

if (!fs.existsSync(filePath)) {
  // 文件不存在，创建一个新文件
  const defaultConfig = {
    downloadPath: downloadValue.value,
    api: [
      {
        name: '服务器',
        upload: 'http://api.silencelurker.xyz/transport/uploadFile',
        download: 'http://api.silencelurker.xyz/transport/getFile'
      }
    ]
  }
  globalConfig.value = defaultConfig
  selectOptions.value = defaultConfig.api
  value.value = selectOptions.value[0].name
  currentUploadUrl.value = selectOptions.value[0].upload
  fs.writeFileSync(filePath, JSON.stringify(defaultConfig, null, 2), 'utf8', (err) => {
    if (err) {
      console.error(err)
    } else {
      console.log('File created successfully with default config')
    }
  })
} else {
  // 文件存在，读取配置
  fs.readFile(filePath, 'utf8', (err, data) => {
    if (err) {
      console.error(err)
      return
    }

    try {
      const config = JSON.parse(data)
      console.log(config)
      globalConfig.value = config
      selectOptions.value = config.api
      value.value = selectOptions.value[0].name
      currentUploadUrl.value = selectOptions.value[0].upload
    } catch (e) {
      console.error('Error parsing JSON', e)
    }
  })
}

const handleDragOver = (event) => {
  event.preventDefault()
}

const handleUpload = (event) => {
  event.preventDefault()
  let file = event.dataTransfer.files[0]
  console.log(`${currentUploadUrl.value}`)
  axios
    .post(
      `${currentUploadUrl.value}`,
      {
        file: file
      },
      {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }
    )
    .then((res) => {
      if (res.status === 200) {
        message.success('上传成功!')
      }
    })
    .catch((err) => {
      console.log(err)
      try {
        if (err.response.status === 400) {
          // message.error('文件已存在!')
          showDeleteConfirm(file)
        }
      } catch (err) {
        message.error('接口异常！')
      }
    })
}

const showDeleteConfirm = (file) => {
  Modal.confirm({
    title: '文件已存在',
    icon: createVNode(ExclamationCircleOutlined),
    content: '您是否同意继续上传并覆盖服务端的文件？',
    okText: '同意',
    okType: 'danger',
    cancelText: '取消',
    onOk() {
      console.log('OK')
      axios
        .post(
          `${currentUploadUrl.value}?replace=true`,
          {
            file: file
          },
          {
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          }
        )
        .then((res) => {
          if (res.status === 200) {
            message.success('覆盖成功!')
          }
        })
    },
    onCancel() {
      console.log('Cancel')
    }
  })
}

watch(value, (_val) => {
  selectOptions.value.forEach((item) => {
    if (item.name === _val) {
      currentUploadUrl.value = item.upload
    }
  })
})
</script>

<style lang="scss" scoped>
@import '../assets/css/global.scss';
.content {
  width: 100%;
  height: 100%;
}
.card {
  position: relative;
  box-sizing: border-box;
  margin: 15px;
  height: calc(100% - 30px);
  width: 100%;
  border-radius: 10px;
  border: 1px dashed rgba(200, 200, 200, 1);
  background-color: rgba(240, 240, 240, 0.5);
  overflow: hidden;
}

.title {
  position: absolute;
  top: 60px;
  font-size: 14px;
  color: #555555;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  font-weight: 600;
  z-index: 50;
}

.circle1 {
  position: absolute;
  height: 300px;
  width: 300px;
  border-radius: 50%;
  opacity: 1;
  background: rgba(121, 72, 234, 0.2);
  backdrop-filter: blur(10px);
  z-index: 100;
  bottom: 40px;
}

.circle2 {
  position: absolute;
  height: 150px;
  width: 150px;
  background-color: #7948ea;
  border-radius: 50%;
  bottom: 115px;
}

.circle3 {
  position: absolute;
  height: 50px;
  width: 50px;
  background-color: #7948ea;
  border-radius: 50%;
  bottom: 165px;
}

.circle4 {
  position: absolute;
  height: 50px;
  width: 50px;
  border: 2px solid #cccccc;
  border-radius: 50%;
  bottom: 165px;
}

.circle {
  position: absolute;
  border: 2px solid rgba(0, 0, 0, 0.1);
  border-radius: 50%;
}

.ping {
  -webkit-animation: ping 0.8s ease-in-out infinite both;
  animation: ping 0.8s ease-in-out infinite both;
}

.pulsate-fwd {
  -webkit-animation: pulsate-fwd 0.5s ease-in-out infinite both;
  animation: pulsate-fwd 0.5s ease-in-out infinite both;
}

/* ----------------------------------------------
 * Generated by Animista on 2023-9-8 3:42:5
 * Licensed under FreeBSD License.
 * See http://animista.net/license for more info.
 * w: http://animista.net, t: @cssanimista
 * ---------------------------------------------- */

/**
 * ----------------------------------------
 * animation ping
 * ----------------------------------------
 */
@-webkit-keyframes ping {
  0% {
    -webkit-transform: scale(0.2);
    transform: scale(0.2);
    opacity: 0.8;
  }
  80% {
    -webkit-transform: scale(1.2);
    transform: scale(1.2);
    opacity: 0;
  }
  100% {
    -webkit-transform: scale(2.2);
    transform: scale(2.2);
    opacity: 0;
  }
}
@keyframes ping {
  0% {
    -webkit-transform: scale(0.2);
    transform: scale(0.2);
    opacity: 0.8;
  }
  80% {
    -webkit-transform: scale(1.2);
    transform: scale(1.2);
    opacity: 0;
  }
  100% {
    -webkit-transform: scale(2.2);
    transform: scale(2.2);
    opacity: 0;
  }
}

/* ----------------------------------------------
 * Generated by Animista on 2023-9-8 4:4:5
 * Licensed under FreeBSD License.
 * See http://animista.net/license for more info.
 * w: http://animista.net, t: @cssanimista
 * ---------------------------------------------- */

/**
 * ----------------------------------------
 * animation pulsate-fwd
 * ----------------------------------------
 */
@-webkit-keyframes pulsate-fwd {
  0% {
    -webkit-transform: scale(1);
    transform: scale(1);
  }
  50% {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }
  100% {
    -webkit-transform: scale(1);
    transform: scale(1);
  }
}
@keyframes pulsate-fwd {
  0% {
    -webkit-transform: scale(1);
    transform: scale(1);
  }
  50% {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }
  100% {
    -webkit-transform: scale(1);
    transform: scale(1);
  }
}
</style>
