<template>
  <section class="layout-list-content padding-top-size-nomal padding-bottom-size-nomal">
    <el-form
      ref="composeForm"
      :rules="rules"
      :model="form"
      label-width="100px"
      style="margin: 0 auto; width: 1000px;"
    >
      <el-form-item label="标题" prop="articleTitle">
        <el-input
          v-model="form.articleTitle"
          type="input"
          autofocus="true"
          placeholder="标题（1~20个字符）"
        />
      </el-form-item>
      <el-form-item label="所属版块" prop="plateId">
        <el-select v-model="form.plateId" placeholder="所属版块" style="width: 100%;">
          <el-option
            v-for="item in options"
            :key="item.id"
            :label="item.plateName"
            :value="item.id"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="" prop="articleContent">
        <editor v-model="form.articleContent" />
      </el-form-item>
      <el-form-item label="相关图片">
        <fileUp v-model="form.imgFileIds" accept="image/*" accept-tips="只能上传图片格式的文件" dir="dir" />
      </el-form-item>
      <el-form-item label="匿名发帖">
        <el-checkbox
          v-model="checked"
          @change="changeAnonymous"
        />
      </el-form-item>
    </el-form>
  </section>
</template>
<script>
import { getUserInfo, createArticle, updateArticle, getNoPubArticle, getPlateList } from '@/api/index'
import fileUp from '@/component/fileUp'
import editor from '@/component/editor'
function nameValidator (rule, value, callback) {
  if (value.trim() === '') {
    callback(new Error('标题不能为空'))
  } else {
    callback()
  }
}
export default {
  components: {
    fileUp,
    editor
  },
  props: {
    flag: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      checked: false,
      domainId: '',
      form: {
        id: '',
        articleTitle: '',
        plateId: '',
        articleContent: '',
        imgFileIds: '',
        fileIds: '',
        anonymous: 0,
        articleIsPub: 1
      },
      options: [],
      rules: {
        articleTitle: [
          { required: true, message: '标题不能为空', trigger: 'blur' },
          { min: 1, max: 30, message: '长度在 1 到 30 个字符', trigger: 'blur' },
          {
            validator: nameValidator
          }
        ],
        plateId: [
          { required: true, message: '请选择所属版块', trigger: 'blur' }
        ],
        articleContent: [
          { required: true, message: '内容不能为空', trigger: 'blur' }
        ]

      }
    }
  },
  watch: {
    $route: function () {
      if (!this.$route.query.id) {
        this.form = {
          id: '',
          articleTitle: '',
          plateId: '',
          articleContent: '',
          imgFileIds: '',
          fileIds: '',
          anonymous: 0,
          articleIsPub: 1
        }
      }
    },
    flag (newval, oldval) {
      if (newval > oldval) {
        this.submitForm('composeForm')
      }
    }
  },
  created: function () {

  },
  mounted: function () {
    getUserInfo().then(res => {
      this.domainId = res.domainId
    })
    getPlateList().then(res => {
      this.options = res
    })
    const id = this.$route.query.id
    getNoPubArticle(id === undefined ? '' : id).then(res => {
      this.form = {
        id: res.id,
        articleTitle: res.articleTitle,
        plateId: res.plateId,
        articleContent: res.articleContent,
        imgFileIds: res.imgFileIds,
        fileIds: res.fileIds,
        anonymous: res.anonymous === null ? 0 : res.anonymous,
        articleIsPub: 1
      }
      this.checked = this.form.anonymous === 1
    })
  },
  methods: {
    changeAnonymous () {
      this.form.anonymous = this.checked ? 1 : 0
    },
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          if (this.$route.query.id) {
            updateArticle(this.form).then(res => {
              this.$message.success('修改成功')
              this.$router.push({ path: '/mfs-forum' })
            })
          } else {
            createArticle(this.form).then(res => {
              // this.$message.success('发布成功').then(() => {
              //   setTimeout(() => {
              //     window.close()
              //   }, 1000)
              // })
              this.$message.success('发布成功')
              this.$router.push({ path: '/mfs-forum' })
            }).catch(err => {
              this.$message.error(err)
            })
          }
        }
      })
    }
  }
}
</script>
<style scoped>
 .el-select-dropdown__item:hover{
  background: #3396fc;
  color: #fff ;
}
</style>
