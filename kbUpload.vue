<template>
<div class="kb-upload">
  <img
    v-if="localUploadType === 'image' && resultImgUrl"
    :src="resultImgUrl"
    class="kb-upload__img" />
  <el-upload
    v-if="localUploadUrl"
    :name="localUploadName"
    :action="localUploadUrl"
    :accept="localAccept"
    :data="localData"
    :auto-upload="localAutoUpload"
    :http-request="localHttpRequest ? localHttpRequest : undefined"
    :headers="localHeaders ? localHeaders : undefined"
    :show-file-list="localShowFileList"
    :limit="localLimit"
    :before-upload="methodHandlerBeforeUpload"
    :on-change="methodHandlerOnChange"
    :on-remove="methodHandlerOnRemove"
    :on-success="methodHandlerOnSuccess"
    :on-exceed="() => fNotify('提示', '超出上传最大限制', 'error')"
    v-bind="$attrs"
    ref="upload" >
    <kb-button
      v-if="!btnDisabled"
      :loading="btnLoading"
      :disabled="btnDisabled"
      type="primary" >
      {{info.uploadBtnTitle || '上传'}}
    </kb-button>
  </el-upload>
  <div v-if="localTip" class="kb-upload__tip"><span>**</span>{{localTip}}</div>
</div>

</template>
<script>
import { fNotify } from '@/utils/formatShortcut';
import kbButton from './kbButton.vue';

export default {
  inheritAttrs: false,
  props: ['info'],
  components: {
    kbButton,
  },
  data() {
    return {
      btnLoading: false,
      btnDisabled: false,
      uploaded: false,
      localUploadUrl: '',
      localUploadType: 'file',
      localUploadName: '',
      localAccept: '',
      localTip: '',
      localData: {},
      localAutoUpload: true,
      localHttpRequest: undefined,
      localHeaders: undefined,
      localShowFileList: false,
      localLimit: undefined,
      result: '',
    };
  },
  computed: {
    resultImgUrl() {
      return this.getPicUrl(this.result);
    },
    acceptRegexp() {
      if (!this.localAccept) return false;
      return new RegExp(this.localAccept.replace(/\./g, '').replace(/,/g, '|'));
    },
  },
  watch: {
    info: {
      handler(newVal) {
        const {
          uploadUrl = '',
          uploadType = 'file',
          uploadName = '',
          accept = '',
          previewImgUrl,
          tip = '',
          data = {},
          autoUpload = true,
          httpRequest = undefined,
          headers = undefined,
          showFileList = false,
          limit = undefined,
        } = newVal;
        this.localUploadUrl = uploadUrl;
        this.localUploadType = uploadType;
        this.localUploadName = uploadName;
        this.localAccept = accept;
        this.localTip = tip;
        this.localData = data;
        this.localAutoUpload = autoUpload;
        this.localHttpRequest = httpRequest;
        this.localHeaders = headers;
        this.localShowFileList = showFileList;
        this.localLimit = limit;
        if (uploadType === 'image' && previewImgUrl && !this.uploaded) {
          this.result = previewImgUrl;
        } else {
          this.result = null;
        }
      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    fNotify,
    getPicUrl(key) {
      if (key) return `${process.env.VUE_APP_IMGURL}/${key}/0`;
      return '';
    },
    methodHandlerBeforeUpload(file) {
      this.btnLoading = true;
      if (this.localUploadType === 'image') {
        return new Promise((res, rej) => {
          const reader = new FileReader();
          reader.onload = (e) => {
            const img = new Image();
            img.onload = () => {
              function checkLimit(target, min, max) {
                if (min > 0 && target < min) return 'min';
                if (max > 0 && target > max) return 'max';
                return false;
              }
              const uploadErrorMsg = [];
              if (checkLimit(img.width, this.info.minWidth, this.info.maxWidth) === 'min') {
                uploadErrorMsg.push('图片不够宽');
              } else if (checkLimit(img.width, this.info.minWidth, this.info.maxWidth) === 'max') {
                uploadErrorMsg.push('图片过宽');
              }
              if (checkLimit(img.height, this.info.minHeight, this.info.maxHeight) === 'min') {
                uploadErrorMsg.push('图片不够高');
              } else if (checkLimit(img.height, this.info.minHeight, this.info.maxHeight) === 'max') {
                uploadErrorMsg.push('图片过高');
              }
              if (checkLimit(file.size, 0, this.info.maxSize) === 'min') {
                uploadErrorMsg.push('档案过小');
              } else if (checkLimit(file.size, 0, this.info.maxSize) === 'max') {
                uploadErrorMsg.push('档案过大');
              }
              if (uploadErrorMsg.length) rej(Error(uploadErrorMsg.toString()));
              res(true);
            };
            img.src = e.target.result;
          };
          reader.readAsDataURL(file);
          // 驗證檔案格式
          if (this.acceptRegexp) {
            if (!this.acceptRegexp.test(file.type)) {
              rej(Error('文件格式有误，不支持此格式'));
            }
          } else if (!/image/gi.test(file.type)) {
            rej(Error('文件格式有误，请上传正确图片格式'));
          }
        }).catch((error) => {
          this.$message({
            message: error.message,
            title: '提示',
            type: 'error',
          });
          this.btnLoading = false;
          return Promise.reject();
        });
      } else if (this.localUploadType === 'file') {
        return new Promise((res, rej) => {
          if (this.info.maxSize && file.size > this.info.maxSize) rej(new Error('档案过大'));
          res(true);
        });
      }
      this.btnLoading = true;
      return false;
    },
    methodHandlerOnChange(file, fileList) {
      this.btnLoading = this.btnLoading && false;
      this.handleBtnDisabled(fileList);
    },
    methodHandlerOnRemove(file, fileList) {
      this.handleBtnDisabled(fileList);
    },
    handleBtnDisabled(fileList) {
      const listLen = fileList.length;
      if (this.localLimit !== undefined && listLen >= this.localLimit) this.btnDisabled = true;
      else this.btnDisabled = false;
    },
    methodHandlerOnSuccess(res) {
      const { fileid, picid } = res;
      this.result = fileid || picid;
      this.$emit('success', fileid || res);
      this.uploaded = true;
    },
    methodResetUploaded() {
      this.uploaded = false;
      const { uploadType, previewImgUrl } = this.info;
      this.$refs.upload.clearFiles();
      if (uploadType === 'image' && previewImgUrl && !this.uploaded) {
        this.result = previewImgUrl;
      } else {
        this.result = null;
      }
      this.btnDisabled = false;
      this.btnLoading = false;
    },
    methodSubmit() {
      this.$refs.upload.submit();
    },
  },
};
</script>

