<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="用户ID" prop="memberId">
      <el-input v-model="dataForm.memberId" placeholder="用户ID"></el-input>
    </el-form-item>
    <el-form-item label="动作类型" prop="action">
      <el-input v-model="dataForm.action" placeholder="动作类型"></el-input>
    </el-form-item>
    <el-form-item label="得分" prop="point">
      <el-input v-model="dataForm.point" placeholder="得分"></el-input>
    </el-form-item>
    <el-form-item label="关联的帖子ID" prop="articleId">
      <el-input v-model="dataForm.articleId" placeholder="关联的帖子ID"></el-input>
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
          memberId: '',
          action: '',
          point: '',
          articleId: '',
          commentId: '',
          status: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          memberId: [
            { required: true, message: '用户ID不能为空', trigger: 'blur' }
          ],
          action: [
            { required: true, message: '动作类型不能为空', trigger: 'blur' }
          ],
          point: [
            { required: true, message: '得分不能为空', trigger: 'blur' }
          ],
          articleId: [
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
              url: this.$http.adornUrl(`/api/memberaction/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.memberId = data.memberAction.memberId
                this.dataForm.action = data.memberAction.action
                this.dataForm.point = data.memberAction.point
                this.dataForm.articleId = data.memberAction.articleId
                this.dataForm.commentId = data.memberAction.commentId
                this.dataForm.status = data.memberAction.status
                this.dataForm.createTime = data.memberAction.createTime
                this.dataForm.updateTime = data.memberAction.updateTime
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
              url: this.$http.adornUrl(`/api/memberaction/${!this.dataForm.id ? 'save' : 'update'}`),
              method: `${!this.dataForm.id ? 'post' : 'put'}`,
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'memberId': this.dataForm.memberId,
                'action': this.dataForm.action,
                'point': this.dataForm.point,
                'articleId': this.dataForm.articleId,
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
                  duration: 1500,
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
