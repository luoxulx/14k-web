<template>
  <div class="upload-container">
    <el-upload
      ref="qiniuUpload"
      class="image-uploader"
      :cdnhost="cdnHost"
      :action="uploadUri"
      :limit="1"
      :auto-upload="true"
      :data="extra"
      list-type="picture-card"
      :on-success="picUploadSuccess"
      :on-change="picOnchange"
      :before-upload="picBeforeUpload"
      :on-preview="picUploadPreview"
      :on-remove="picClickRemove">
      <i class="el-icon-plus" />
    </el-upload>
    <el-input v-model="imageUrl" type="text" value="" style="max-width: 80%;" />&nbsp;<el-button size="mini" type="primary" @click="submitUpload">上传</el-button>
    <el-dialog :visible.sync="picDialogVisible">
      <img width="100%" :src="imageUrl" alt="">
    </el-dialog>
  </div>
</template>

<script>
import { deleteQiniuFile, uploadImageToken, saveFileInfo } from '@/api'
import { parseFileSize } from '@/utils'
export default {
  name: 'PicUpload',
  props: {
    value: {
      type: String,
      default: ''
    },
    cdnHost: {
      type: String,
      default: 'https://cn.lnmpa.top/'
    },
    uploadUri: {
      // uploadUri: 'https://upload-na0.qiniup.com', // web 直传地址,不同区域地址也不同
      // uploadUri: 'https://upload.qiniup.com', // web 直传地址,不同区域地址也不同
      type: String,
      default: 'https://upload.qiniup.com'
    }
  },
  data() {
    return {
      picDialogVisible: false,
      accept: [],
      extra: {
        original_name: '',
        key: 'thumbnail', // 默认thumbnail
        token: ''
      },
      picPath: {}, // 上传成功后，供删除按钮使用
      picMime: ''
    }
  },
  computed: {
    imageUrl() {
      if (this.value == null) {
        return ''
      }
      return this.value
    }
  },
  methods: {
    submitUpload() {
      this.$refs.qiniuUpload.submit()
    },
    emitInput(val) {
      this.$emit('input', val)
    },
    picClickRemove(file, fileList) {
      console.log(file, fileList)
      deleteQiniuFile({ key: this.picPath }).then(response => {
        console.log(response)
      })
      this.emitInput('')
    },
    picUploadSuccess(res, file, fileList) {
      const tempData = {
        path: res.key,
        original_name: file.name,
        url: this.cdnHost + res.key + '-14k', // 到底要不要水印？
        // url: this.cdnHost + res.key,
        size: parseFileSize(file.size),
        mime: this.picMime,
        hash: res.hash
      }

      this.picPath = tempData.path
      this.emitInput(tempData.url)

      saveFileInfo(tempData).then(res2 => {
        console.info('Upload Info Save Successful. ', res2)
      })
      // 文件信息保存成功，清空extra的key
      this.extra.key = 'thumbnail'
    },
    picUploadPreview(file) {
      console.log(file)
      this.picDialogVisible = true
    },
    picOnchange(file, fileList) {
      this.extra.original_name = file.name
      this.picMime = file.raw.type || ''

      // uploadImageToken(this.extra).then(res => {
      //   this.extra.token = res.data.token
      //   this.extra.key = res.data.key
      // })
    },
    picBeforeUpload(file) {
      this.picMime = file.type || ''
      // 自动上传前需要拿到被保存的图片信息
      // console.log(file)
      const tempData = {
        original_name: '',
        key: ''
      }
      uploadImageToken(tempData).then(res => {
        this.extra.token = res.data.token
        this.extra.key = res.data.key
      })
    }
  }
}
</script>

<style>
  .el-upload {
    border: 1px dotted rebeccapurple;
    max-width: 75px;
    max-height: 75px;
    line-height: 75px;
  }
</style>
