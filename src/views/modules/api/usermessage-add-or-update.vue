<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="发送消息的用户ID" prop="fromUserId">
      <el-input v-model="dataForm.fromUserId" placeholder="发送消息的用户ID"></el-input>
    </el-form-item>
    <el-form-item label="接收消息的用户ID" prop="toUserId">
      <el-input v-model="dataForm.toUserId" placeholder="接收消息的用户ID"></el-input>
    </el-form-item>
    <el-form-item label="消息可能关联的帖子" prop="postId">
      <el-input v-model="dataForm.postId" placeholder="消息可能关联的帖子"></el-input>
    </el-form-item>
    <el-form-item label="消息可能关联的评论" prop="commentId">
      <el-input v-model="dataForm.commentId" placeholder="消息可能关联的评论"></el-input>
    </el-form-item>
    <el-form-item label="消息内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="消息内容"></el-input>
    </el-form-item>
    <el-form-item label="消息类型（0：系统消息，1：回复文章，2：回复评论）" prop="type">
      <el-input v-model="dataForm.type" placeholder="消息类型（0：系统消息，1：回复文章，2：回复评论）"></el-input>
    </el-form-item>
    <el-form-item label="消息状态（0：未读，1：已读）" prop="status">
      <el-input v-model="dataForm.status" placeholder="消息状态（0：未读，1：已读）"></el-input>
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
          fromUserId: '',
          toUserId: '',
          postId: '',
          commentId: '',
          content: '',
          type: '',
          status: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          fromUserId: [
            { required: true, message: '发送消息的用户ID不能为空', trigger: 'blur' }
          ],
          toUserId: [
            { required: true, message: '接收消息的用户ID不能为空', trigger: 'blur' }
          ],
          postId: [
            { required: true, message: '消息可能关联的帖子不能为空', trigger: 'blur' }
          ],
          commentId: [
            { required: true, message: '消息可能关联的评论不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '消息内容不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '消息类型（0：系统消息，1：回复文章，2：回复评论）不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '消息状态（0：未读，1：已读）不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/api/usermessage/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.fromUserId = data.userMessage.fromUserId
                this.dataForm.toUserId = data.userMessage.toUserId
                this.dataForm.postId = data.userMessage.postId
                this.dataForm.commentId = data.userMessage.commentId
                this.dataForm.content = data.userMessage.content
                this.dataForm.type = data.userMessage.type
                this.dataForm.status = data.userMessage.status
                this.dataForm.createTime = data.userMessage.createTime
                this.dataForm.updateTime = data.userMessage.updateTime
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
              url: this.$http.adornUrl(`/api/usermessage/${!this.dataForm.id ? 'save' : 'update'}`),
              method: `${!this.dataForm.brandId ? 'post' : 'put'}`,
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'fromUserId': this.dataForm.fromUserId,
                'toUserId': this.dataForm.toUserId,
                'postId': this.dataForm.postId,
                'commentId': this.dataForm.commentId,
                'content': this.dataForm.content,
                'type': this.dataForm.type,
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
