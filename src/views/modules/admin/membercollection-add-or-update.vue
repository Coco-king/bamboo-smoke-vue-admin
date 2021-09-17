<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="用户ID" prop="memberId">
        <el-input v-model="dataForm.memberId" placeholder="用户ID"></el-input>
      </el-form-item>
      <el-form-item label="文章ID" prop="articleId">
        <el-input v-model="dataForm.articleId" placeholder="文章ID"></el-input>
      </el-form-item>
      <el-form-item label="文章所属的用户ID" prop="articleMemberId">
        <el-input
          v-model="dataForm.articleMemberId"
          placeholder="文章所属的用户ID"
        ></el-input>
      </el-form-item>
      <el-form-item label="消息状态" prop="status">
        <el-input v-model="dataForm.status" placeholder="消息状态"></el-input>
      </el-form-item>
      <el-form-item label="创建时间" prop="createTime">
        <el-input
          v-model="dataForm.createTime"
          placeholder="创建时间"
        ></el-input>
      </el-form-item>
      <el-form-item label="修改时间" prop="updateTime">
        <el-input
          v-model="dataForm.updateTime"
          placeholder="修改时间"
        ></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        memberId: '',
        articleId: '',
        articleMemberId: '',
        status: '',
        createTime: '',
        updateTime: ''
      },
      dataRule: {
        memberId: [
          {required: true, message: '用户ID不能为空', trigger: 'blur'}
        ],
        articleId: [
          {required: true, message: '文章ID不能为空', trigger: 'blur'}
        ],
        articleMemberId: [
          {
            required: true,
            message: '文章所属的用户ID不能为空',
            trigger: 'blur'
          }
        ],
        status: [
          {required: true, message: '消息状态不能为空', trigger: 'blur'}
        ],
        createTime: [
          {required: true, message: '创建时间不能为空', trigger: 'blur'}
        ],
        updateTime: [
          {required: true, message: '修改时间不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/membercollection/info/${this.dataForm.id}`
            ),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.memberId = data.memberCollection.memberId
              this.dataForm.articleId = data.memberCollection.articleId
              this.dataForm.articleMemberId =
                data.memberCollection.articleMemberId
              this.dataForm.status = data.memberCollection.status
              this.dataForm.createTime = data.memberCollection.createTime
              this.dataForm.updateTime = data.memberCollection.updateTime
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
              `/admin/membercollection/${!this.dataForm.id ? 'save' : 'update'}`
            ),
            method: `${!this.dataForm.id ? 'post' : 'put'}`,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              memberId: this.dataForm.memberId,
              articleId: this.dataForm.articleId,
              articleMemberId: this.dataForm.articleMemberId,
              status: this.dataForm.status,
              createTime: this.dataForm.createTime,
              updateTime: this.dataForm.updateTime
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1000,
                onClose: () => {
                  this.visible = false
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
  }
}
</script>
