<template>
  <div class="content2">
    <div class="title2">设置</div>
    <a-divider />
    <div class="settings">
      <a-collapse v-model:activeKey="activeKey" class="custom-collapse">
        <a-collapse-panel key="1" header="系统设置" class="user-select">
          <div style="display: flex; flex-direction: row; align-items: center">
            <p style="white-space: nowrap">默认下载路径：</p>
            <a-input-search
              v-model:value="downloadValue"
              enter-button="浏览"
              style="height: 32px; padding-left: 10px"
              @search="selectDirectory"
            />
          </div>
        </a-collapse-panel>
        <a-collapse-panel key="2" header="接口设置" class="user-select">
          <div style="display: flex; flex-direction: column">
            <!-- 添加接口 -->
            <div
              class="add"
              style="
                display: flex;
                flex-direction: row;
                align-items: center;
                justify-content: space-between;
              "
            >
              <div style="display: flex; flex-direction: row; align-items: center">
                <p style="white-space: nowrap">接口名称：</p>
                <a-input
                  v-model:value="nameValue"
                  style="height: 32px; padding-left: 10px; width: 180px"
                  placeholder="请输入接口名称"
                />
              </div>

              <a-button
                type="primary"
                style="margin-left: 20px; width: 80px"
                :class="addButtonClass"
                @click="handleAdd"
                >添加</a-button
              >
            </div>

            <br />
            <div class="url" style="display: flex; flex-direction: column">
              <a-input v-model:value="uploadUrlValue" placeholder="上传接口地址">
                <template #addonBefore>
                  <a-select v-model:value="uploadHttpValue" style="width: 90px">
                    <a-select-option value="http://">Http://</a-select-option>
                    <a-select-option value="https://">Https://</a-select-option>
                  </a-select>
                </template>
              </a-input>
              <br />
              <a-input v-model:value="downloadUrlValue" placeholder="下载接口地址">
                <template #addonBefore>
                  <a-select v-model:value="downloadHttpValue" style="width: 90px">
                    <a-select-option value="http://">Http://</a-select-option>
                    <a-select-option value="https://">Https://</a-select-option>
                  </a-select>
                </template>
              </a-input>
            </div>
            <!-- 接口列表 -->
            <br />
            <a-divider />
            <br />
            <div>接口列表：</div>
            <br />
            <a-table
              :data-source="dataSource"
              :pagination="false"
              :row-class-name="(_record, index) => (index % 2 === 1 ? 'table-striped' : null)"
            >
              <a-table-column
                key="name"
                title="接口名称"
                data-index="name"
                width="100px"
                ellipsis="true"
              />
              <a-table-column key="upload" title="上传接口" data-index="upload" />
              <a-table-column key="download" title="下载接口" data-index="download" />
              <a-table-column key="action" title="操作" width="80px">
                <template #default="{ index }">
                  <a-popconfirm
                    title="确认删除"
                    ok-text="确认"
                    cancel-text="取消"
                    @confirm="handleDelete(index)"
                  >
                    <a-button type="primary" danger ghost style="width: 50px"> 删除 </a-button>
                  </a-popconfirm>
                </template>
              </a-table-column>
            </a-table>
          </div>
        </a-collapse-panel>
        <a-collapse-panel key="3" header="关于我们" class="user-select">
          <a-qrcode error-level="H" value="https://github.com/JohnsonSii" />
        </a-collapse-panel>
      </a-collapse>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch } from 'vue'
import { message } from 'ant-design-vue'
const { ipcRenderer } = require('electron')

const selectDirectory = () => {
  ipcRenderer.send('open-directory-dialog')
}

ipcRenderer.on('selected-directory', (event, path) => {
  if (path) {
    downloadValue.value = path
    globalConfig.value.downloadPath = downloadValue.value
    fs.writeFileSync(filePath, JSON.stringify(globalConfig.value, null, 2), 'utf8')
  }
})

const fs = require('fs')
const os = require('os')
const path = require('path')
const addButtonClass = ref('')

const downloadsFolder = path.join(os.homedir(), 'Downloads')
const downloadValue = ref(downloadsFolder)

const nameValue = ref('')
// 上传接口
const uploadUrlValue = ref('')
const uploadHttpValue = ref('http://')
// 下载接口
const downloadUrlValue = ref('')
const downloadHttpValue = ref('http://')

const dataSource = reactive([])
const filePath = './config.json'
const globalConfig = ref()

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
  dataSource.splice(0, dataSource.length, ...defaultConfig.api)
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
      dataSource.splice(0, dataSource.length, ...config.api)
    } catch (e) {
      console.error('Error parsing JSON', e)
    }
  })
}
const handleDelete = (index) => {
  dataSource.splice(index, 1)
  globalConfig.value.api = dataSource
  fs.writeFileSync(filePath, JSON.stringify(globalConfig.value, null, 2), 'utf8')
}

const handleAdd = () => {
  if (nameValue.value != '' && uploadUrlValue.value != '' && downloadUrlValue.value != '') {
    console.log('新增')
    const currentApiConfig = {
      name: nameValue.value,
      upload: uploadHttpValue.value + uploadUrlValue.value,
      download: downloadHttpValue.value + downloadUrlValue.value
    }
    globalConfig.value.api.push(currentApiConfig)
    dataSource.splice(0, dataSource.length, ...globalConfig.value.api)
    fs.writeFileSync(filePath, JSON.stringify(globalConfig.value, null, 2), 'utf8')
    message.success('接口添加成功！')
    nameValue.value = ''
    uploadUrlValue.value = ''
    downloadUrlValue.value = ''
  } else {
    message.error('接口信息填写不完整')
    addButtonClass.value = 'shake-horizontal'
    setTimeout(() => {
      addButtonClass.value = ''
    }, 800)
  }
}

const activeKey = ref(['1', '2'])
watch(activeKey, (val) => {
  console.log(val)
})
</script>

<style lang="scss" scoped>
@import '../assets/css/global.scss';
.content2 {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}
.title2 {
  font-size: 18px;
  color: #555555;
  margin: 25px 0;
  font-weight: 600;
}
.settings {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: auto;
  justify-content: flex-start;
  align-items: center;
}

.custom-collapse {
  width: calc(100% - 70px);
  margin: 35px 0;
}

.user-select {
  user-select: none;
}

.shake-horizontal {
  -webkit-animation: shake-horizontal 0.8s cubic-bezier(0.455, 0.03, 0.515, 0.955) both;
  animation: shake-horizontal 0.8s cubic-bezier(0.455, 0.03, 0.515, 0.955) both;
}

/* ----------------------------------------------
 * Generated by Animista on 2023-9-8 9:10:35
 * Licensed under FreeBSD License.
 * See http://animista.net/license for more info.
 * w: http://animista.net, t: @cssanimista
 * ---------------------------------------------- */

/**
 * ----------------------------------------
 * animation shake-horizontal
 * ----------------------------------------
 */
@-webkit-keyframes shake-horizontal {
  0%,
  100% {
    -webkit-transform: translateX(0);
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70% {
    -webkit-transform: translateX(-10px);
    transform: translateX(-10px);
  }
  20%,
  40%,
  60% {
    -webkit-transform: translateX(10px);
    transform: translateX(10px);
  }
  80% {
    -webkit-transform: translateX(8px);
    transform: translateX(8px);
  }
  90% {
    -webkit-transform: translateX(-8px);
    transform: translateX(-8px);
  }
}
@keyframes shake-horizontal {
  0%,
  100% {
    -webkit-transform: translateX(0);
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70% {
    -webkit-transform: translateX(-10px);
    transform: translateX(-10px);
  }
  20%,
  40%,
  60% {
    -webkit-transform: translateX(10px);
    transform: translateX(10px);
  }
  80% {
    -webkit-transform: translateX(8px);
    transform: translateX(8px);
  }
  90% {
    -webkit-transform: translateX(-8px);
    transform: translateX(-8px);
  }
}
</style>
