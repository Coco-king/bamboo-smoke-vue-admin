<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="用户ID" prop="userId">
      <el-input v-model="dataForm.userId" placeholder="用户ID"></el-input>
    </el-form-item>
    <el-form-item label="动作类型" prop="action">
      <el-input v-model="dataForm.action" placeholder="动作类型"></el-input>
    </el-form-item>
    <el-form-item label="得分" prop="point">
      <el-input v-model="dataForm.point" placeholder="得分"></el-input>
    </el-form-item>
    <el-form-item label="关联的帖子ID" prop="postId">
      <el-input v-model="dataForm.postId" placeholder="关联的帖子ID"></el-input>
    </el-form-item>
    <el-form-item label="关联的评论ID" prop="commentId">
      <el-input v-model="dataForm.commentId" placeholder="关联的评论ID"></el-input>
    </el-form-item>
    <el-form-item label="状态" prop="status">
      <el-input v-model="dataForm.status" placeholder="状态"></el-input>
    </el-form-item>
    <el-form-item label="创建时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
    </el-form-item>
    <el-form-item label="修改时间" prop="updateTime">
      <el-input v-model="dataForm.updateTime" placeholder="修改时间"></el-input>
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
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          userId: '',
          action: '',
          point: '',
          postId: '',
          commentId: '',
          status: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          userId: [
            { required: true, message: '用户ID不能为空', trigger: 'blur' }
          ],
          action: [
            { required: true, message: '动作类型不能为空', trigger: 'blur' }
          ],
          point: [
            { required: true, message: '得分不能为空', trigger: 'blur' }
          ],
          postId: [
            { required: true, message: '关联的帖子ID不能为空', trigger: 'blur' }
          ],
          commentId: [
            { required: true, message: '关联的评论ID不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '状态不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
          ],
          updateTime: [
            { required: true, message: '修改时间不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/api/useraction/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userId = data.userAction.userId
                this.dataForm.action = data.userAction.action
                this.dataForm.point = data.userAction.point
                this.dataForm.postId = data.userAction.postId
                this.dataForm.commentId = data.userAction.commentId
                this.dataForm.status = data.userAction.status
                this.dataForm.createTime = data.userAction.createTime
                this.dataForm.updateTime = data.userAction.updateTime
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/api/useraction/${!this.dataForm.id ? 'save' : 'update'}`),
              method: `${!this.dataForm.brandId ? 'post' : 'put'}`,
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userId': this.dataForm.userId,
                'action': this.dataForm.action,
                'point': this.dataForm.point,
                'postId': this.dataForm.postId,
                'commentId': this.dataForm.commentId,
                'status': this.dataForm.status,
                'createTime': this.dataForm.createTime,
                'updateTime': this.dataForm.updateTime
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
