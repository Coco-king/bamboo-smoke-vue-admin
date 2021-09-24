<template>
  <el-dialog
    width="60%"
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-steps :active="stepActive" style="margin-bottom: 2em" align-center>
      <el-step title="设置文章参数" icon="el-icon-setting"></el-step>
      <el-step title="上传文章封面" icon="el-icon-picture"></el-step>
      <el-step title="编写文章内容" icon="el-icon-edit"></el-step>
    </el-steps>
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      label-width="80px"
    >
      <div v-show="stepActive === 1">
        <el-row>
          <el-col :span="23">
            <el-form-item label="标题" prop="title">
              <el-input v-model="dataForm.title" placeholder="标题"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11">
            <el-form-item label="分类" prop="categoryId">
              <category-select v-model="dataForm.categoryId"></category-select>
            </el-form-item>
          </el-col>
          <el-col :span="1">&nbsp;</el-col>
          <el-col :span="11">
            <el-form-item label="编辑模式" prop="editMode">
              <el-radio-group v-model="dataForm.editMode">
                <el-radio label="0" border>Markdown</el-radio>
                <el-radio label="1" border>HTML可视化</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11">
            <el-form-item label="所属用户" prop="memberId">
              <member-select ref="memberSelect" v-model="dataForm.memberId"></member-select>
            </el-form-item>
          </el-col>
          <el-col :span="1">&nbsp;</el-col>
          <el-col :span="11">
            <el-form-item label="置顶等级" prop="level">
              <el-input v-model="dataForm.level" placeholder="置顶等级"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11">
            <el-form-item label="点赞人数" prop="voteUp">
              <el-input v-model="dataForm.voteUp" placeholder="点赞人数"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="1">&nbsp;</el-col>
          <el-col :span="11">
            <el-form-item label="点踩人数" prop="voteDown">
              <el-input v-model="dataForm.voteDown" placeholder="点踩人数"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11">
            <el-form-item label="访问量" prop="viewCount">
              <el-input v-model="dataForm.viewCount" placeholder="访问量"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="1">&nbsp;</el-col>
          <el-col :span="11">
            <el-form-item label="评论数量" prop="commentCount">
              <el-input
                v-model="dataForm.commentCount"
                placeholder="评论数量"
              ></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="11">
            <el-form-item label="状态" prop="status">
              <el-select v-model="dataForm.status" clearable placeholder="请选择状态" style="width: 100%;">
                <el-option
                  v-for="item in options"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="1">&nbsp;</el-col>
          <el-col :span="11">
            <el-form-item label="是否精华" prop="recommend">
              <el-radio-group v-model="dataForm.recommend">
                <el-radio :label="false" border>否</el-radio>
                <el-radio :label="true" border>是</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
      </div>
      <div v-show="stepActive === 2">
        <el-row>
          <el-col :span="23">
            <el-form-item label="封面" prop="coverImage">
              <el-upload
                :data="dataObj"
                class="avatar-upload"
                action="https://bamboo-smoke-img.oss-cn-shanghai.aliyuncs.com"
                :show-file-list="false"
                :on-success="handleAvatarSuccess"
                :before-upload="beforeAvatarUpload"
                :on-remove="handleRemove"
              >
                <img
                  alt="封面"
                  style="object-fit: contain"
                  v-if="dataForm.coverImage"
                  :src="dataForm.coverImage"
                  class="avatar"
                >
                <i v-else class="el-icon-plus avatar-upload-icon"></i>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过5m，不上传请直接选择下一步</div>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>
      </div>
      <div v-show="stepActive === 3">
        <el-form-item label="内容" prop="content">
          <div v-if="dataForm.editMode === '1'">
            <article-html-editor
              v-model="dataForm.content"
              ref="articleHtmlEditor"
              type="article-upload"
            ></article-html-editor>
          </div>
        </el-form-item>
      </div>
    </el-form>
    <span slot="footer" class="dialog-footer" style="text-align: right;margin-top: 1em;">
      <el-button @click="visible = false">取消</el-button>
      <el-button :disabled="stepActive <= 1" @click="prev">上一步</el-button>
      <el-button v-if="stepActive !== 3" @click="next" type="primary">下一步</el-button>
      <el-button v-if="stepActive === 3" @click="clearEditor" type="danger">清空</el-button>
      <el-button v-if="stepActive === 3" @click=" dataFormSubmit()" type="success">提交</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { policy } from '@/components/upload/policy'
import { getUUID } from '@/utils'
import categorySelect from '@/components/category-select'
import articleHtmlEditor from '@/components/article-html-editor'
import memberSelect from '@/components/member-select'

export default {
  components: {categorySelect, articleHtmlEditor, memberSelect},
  data() {
    return {
      stepActive: 1,
      visible: false,
      dataObj: {},
      categoryList: [],
      dataForm: {
        id: 0,
        title: '',
        coverImage: '',
        content: '',
        editMode: '0',
        categoryId: '',
        memberId: '',
        voteUp: '0',
        voteDown: '0',
        viewCount: '0',
        commentCount: '0',
        recommend: false,
        level: '0',
        status: '0'
      },
      dataRule: {
        title: [{required: true, message: '标题不能为空', trigger: 'blur'}],
        editMode: [
          {
            required: true,
            message: '编辑模式：html可视化，markdown ..不能为空',
            trigger: 'blur'
          }
        ],
        categoryId: [
          {required: true, message: '分类不能为空', trigger: 'change'}
        ],
        memberId: [
          {required: true, message: '创建人不能为空', trigger: 'blur'}
        ],
        voteUp: [
          {required: true, message: '点赞人数不能为空', trigger: 'blur'}
        ],
        voteDown: [
          {required: true, message: '点踩人数不能为空', trigger: 'blur'}
        ],
        viewCount: [
          {required: true, message: '访问量不能为空', trigger: 'blur'}
        ],
        commentCount: [
          {required: true, message: '评论数量不能为空', trigger: 'blur'}
        ],
        recommend: [
          {required: true, message: '是否为精华不能为空', trigger: 'blur'}
        ],
        level: [
          {required: true, message: '置顶等级不能为空', trigger: 'blur'}
        ],
        status: [
          {
            required: true,
            message: '状态（0：审核未通过 1：审核通过）不能为空',
            trigger: 'blur'
          }
        ]
      },
      options: [
        {
          value: '0',
          label: '审核中',
          type: 'warning'
        },
        {
          value: '1',
          label: '审核通过',
          type: 'success'
        },
        {
          value: '-1',
          label: '审核不通过',
          type: 'danger'
        }
      ]
    }
  },
  methods: {
    clearEditor() {
      this.$refs.articleHtmlEditor.clearContent()
    },
    next() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          this.stepActive++
          if (this.stepActive > 2) {
            this.dataRule.content = [{required: true, message: '内容不能为空', trigger: 'blur'}]
          }
        }
      })
    },
    prev() {
      this.stepActive--
      if (this.stepActive < 3) {
        this.dataRule.content = []
      }
    },
    beforeAvatarUpload(file) {
      const isImg = file.type === 'image/jpeg' || file.type === 'image/png'
      const isLt5M = file.size / 1024 / 1024 < 5

      if (!isImg) {
        this.$message.error('上传头像图片只能是 JPG/PNG 格式！')
      }

      if (!isLt5M) {
        this.$message.error('上传头像图片大小不能超过 5MB！')
      }

      if (isImg && isLt5M) {
        let _self = this
        return new Promise((resolve, reject) => {
          policy('article-cover')
          .then(response => {
            _self.dataObj.policy = response.data.policy
            _self.dataObj.signature = response.data.signature
            _self.dataObj.ossaccessKeyId = response.data.accessId
            _self.dataObj.key = response.data.dir + getUUID() + '_${filename}'
            _self.dataObj.dir = response.data.dir
            _self.dataObj.host = response.data.host
            resolve(true)
          })
          .catch(err => {
            reject(err)
          })
        })
      } else {
        return false
      }
    },
    handleAvatarSuccess(res, file) {
      this.dataForm.coverImage = this.dataObj.host + '/' + this.dataObj.key.replace('${filename}', file.name)
    },
    handleRemove(file) {
      this.$http({
        url: this.$http.adornUrl('/api/oss/remove'),
        method: 'delete',
        data: this.$http.adornData({urls: [file.url]}, false)
      }).then(res => {
        if (res && res.code === 0) {
          this.dataForm.coverImage = ''
        }
      })
    },
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.stepActive = 1
      this.dataRule.content = []
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/article/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.title = data.article.title
              this.dataForm.coverImage = data.article.coverImage
              this.dataForm.content = data.article.content
              this.dataForm.editMode = data.article.editMode
              this.dataForm.categoryId = data.article.categoryId
              this.dataForm.memberId = data.article.memberId
              this.dataForm.voteUp = data.article.voteUp
              this.dataForm.voteDown = data.article.voteDown
              this.dataForm.viewCount = data.article.viewCount
              this.dataForm.commentCount = data.article.commentCount
              this.dataForm.recommend = data.article.recommend
              this.dataForm.level = data.article.level
              this.dataForm.status = data.article.status
              this.$refs['memberSelect'].init(this.dataForm.memberId)
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/article/${!this.dataForm.id ? 'save' : 'update'}`
            ),
            method: `${!this.dataForm.id ? 'post' : 'put'}`,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              title: this.dataForm.title,
              coverImage: this.dataForm.coverImage,
              content: this.dataForm.content,
              editMode: this.dataForm.editMode,
              categoryId: this.dataForm.categoryId,
              memberId: this.dataForm.memberId,
              voteUp: this.dataForm.voteUp,
              voteDown: this.dataForm.voteDown,
              viewCount: this.dataForm.viewCount,
              commentCount: this.dataForm.commentCount,
              recommend: this.dataForm.recommend,
              level: this.dataForm.level,
              status: this.dataForm.status
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1000,
                onClose: () => {
                  this.visible = false
                  this.stepActive = 1
                  this.dataRule.content = []
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      })
    }
  },
  watch: {
    visible(newVal) {
      if (newVal === false) {
        this.$refs['memberSelect'].closeSelect()
      }
    }
  }
}
</script>

<style>
.avatar-upload .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  width: 100%;
}

.avatar-upload .el-upload:hover {
  border-color: #409EFF;
}

.avatar-upload-icon {
  font-size: 28px;
  color: #8c939d;
  width: 100%;
  height: 300px;
  line-height: 300px;
  text-align: center;
}

.avatar {
  width: 100%;
  height: 300px;
  display: block;
}
</style>
