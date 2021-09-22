<template>
  <div class="mod-demo-ueditor">
    <script :id="ueId" class="ueditor-box" type="text/plain" style="width: 100%; height: 260px;">hello world!</script>
    <!-- 获取内容 -->
    <p>
      <el-button @click="getContent()">获得内容</el-button>
    </p>
  </div>
</template>

<script>
import ueditor from 'ueditor'

export default {
  name: 'articleHtmlUEditor',
  props: {
    value: String,
    type: String
  },
  data() {
    return {
      ue: null,
      ueId: `J_ueditorBox_${new Date().getTime()}`,
      ueContent: ''
    }
  },
  mounted() {
    this.ue = ueditor.getEditor(this.ueId, {
      serverUrl: `${window.SITE_CONFIG.baseUrl}/api/oss/upload/${this.type}`, // 服务器统一请求接口路径
      zIndex: 3000
    })
  },
  methods: {
    getContent() {
      this.ue.ready(() => {
        this.ueContent = this.ue.getContent()
        console.log(this.ueContent)
      })
    }
  }
}
</script>

<style lang="scss">
.mod-demo-ueditor {
  position: relative;
  z-index: 510;

  > .el-alert {
    margin-bottom: 10px;
  }
}
</style>
