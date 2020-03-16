<template>
  <div class="app-container">
    <el-container>
      <el-header>
        <el-button type="primary" size="mini">
          <router-link :to="{path: '/administration/file'}">返回列表</router-link>
        </el-button>
      </el-header>
      <el-main>
        <el-upload
          class="upload-demo"
          action="https://jsonplaceholder.typicode.com/posts/"
          drag
          multiple
          :on-preview="clickOneFileToPreview"
          :on-remove="clickOneFileToRemove"
          :on-success="onSuccess"
          :on-error="onError"
          :auto-upload="false"
        >
          <i class="el-icon-upload" />
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        </el-upload>
        <el-tag size="mini" type="danger" closable>完整路径包含前缀目录，eg: A/B/C/xxx.tar.gz</el-tag>
      </el-main>
      <el-footer>
        <el-button v-loading="submitLoading" @click="submitfileForm">保存上传结果</el-button>
      </el-footer>
    </el-container>
  </div>
</template>

<script>
/* 需求：
0 上传前统一一个弹窗，要求输入统一的前缀目录
1 多选拖拽；
2 web直传；
3 批量保存结果至MySQL
4 动态展示结果
5 所有点击操作时触发钩子
*/
import { fileCreate } from '@/api'
export default {
  name: 'CreateFile',
  components: {},
  data() {
    return {
      submitLoading: false,
      submitLoadingText: '数据提交中...',
      fileForm: {}
    }
  },
  methods: {
    submitfileForm() {
      this.submitLoading = true
      fileCreate(this.fileForm).then(response => {
        if (response.status === true) {
          this.$message.success('create successful')
          this.$router.push('/file/index')
        } else {
          return false
        }
      })
      this.submitLoading = false
    },
    clickOneFileToPreview(file) {
      console.log(file)
    },
    clickOneFileToRemove(file, fileList) {
      console.log(file)
      console.log(fileList)
    },
    onSuccess(response, file, fileList) {
      console.log(response)
      console.log(file)
      console.log(fileList)
    },
    onError(err, file, fileList) {
      console.log(err)
      console.log(file)
      console.log(fileList)
    }
  }
}
</script>
