<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="评论的内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="评论的内容"></el-input>
    </el-form-item>
    <el-form-item label="回复的评论ID" prop="parentId">
      <el-input v-model="dataForm.parentId" placeholder="回复的评论ID"></el-input>
    </el-form-item>
    <el-form-item label="评论的内容ID" prop="postId">
      <el-input v-model="dataForm.postId" placeholder="评论的内容ID"></el-input>
    </el-form-item>
    <el-form-item label="评论的用户ID" prop="userId">
      <el-input v-model="dataForm.userId" placeholder="评论的用户ID"></el-input>
    </el-form-item>
    <el-form-item label="用户认证标识" prop="userAuthName">
      <el-input v-model="dataForm.userAuthName" placeholder="用户认证标识"></el-input>
    </el-form-item>
    <el-form-item label="“顶”的数量" prop="voteUp">
      <el-input v-model="dataForm.voteUp" placeholder="“顶”的数量"></el-input>
    </el-form-item>
    <el-form-item label="“踩”的数量" prop="voteDown">
      <el-input v-model="dataForm.voteDown" placeholder="“踩”的数量"></el-input>
    </el-form-item>
    <el-form-item label="置顶等级" prop="level">
      <el-input v-model="dataForm.level" placeholder="置顶等级"></el-input>
    </el-form-item>
    <el-form-item label="评论的状态" prop="status">
      <el-input v-model="dataForm.status" placeholder="评论的状态"></el-input>
    </el-form-item>
    <el-form-item label="创建时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
    </el-form-item>
    <el-form-item label="更新时间" prop="updateTime">
      <el-input v-model="dataForm.updateTime" placeholder="更新时间"></el-input>
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
          content: '',
          parentId: '',
          postId: '',
          userId: '',
          userAuthName: '',
          voteUp: '',
          voteDown: '',
          level: '',
          status: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          content: [
            { required: true, message: '评论的内容不能为空', trigger: 'blur' }
          ],
          parentId: [
            { required: true, message: '回复的评论ID不能为空', trigger: 'blur' }
          ],
          postId: [
            { required: true, message: '评论的内容ID不能为空', trigger: 'blur' }
          ],
          userId: [
            { required: true, message: '评论的用户ID不能为空', trigger: 'blur' }
          ],
          userAuthName: [
            { required: true, message: '用户认证标识不能为空', trigger: 'blur' }
          ],
          voteUp: [
            { required: true, message: '“顶”的数量不能为空', trigger: 'blur' }
          ],
          voteDown: [
            { required: true, message: '“踩”的数量不能为空', trigger: 'blur' }
          ],
          level: [
            { required: true, message: '置顶等级不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '评论的状态不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建时间不能为空', trigger: 'blur' }
          ],
          updateTime: [
            { required: true, message: '更新时间不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/api/comment/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.content = data.comment.content
                this.dataForm.parentId = data.comment.parentId
                this.dataForm.postId = data.comment.postId
                this.dataForm.userId = data.comment.userId
                this.dataForm.userAuthName = data.comment.userAuthName
                this.dataForm.voteUp = data.comment.voteUp
                this.dataForm.voteDown = data.comment.voteDown
                this.dataForm.level = data.comment.level
                this.dataForm.status = data.comment.status
                this.dataForm.createTime = data.comment.createTime
                this.dataForm.updateTime = data.comment.updateTime
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
              url: this.$http.adornUrl(`/api/comment/${!this.dataForm.id ? 'save' : 'update'}`),
              method: `${!this.dataForm.brandId ? 'post' : 'put'}`,
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'content': this.dataForm.content,
                'parentId': this.dataForm.parentId,
                'postId': this.dataForm.postId,
                'userId': this.dataForm.userId,
                'userAuthName': this.dataForm.userAuthName,
                'voteUp': this.dataForm.voteUp,
                'voteDown': this.dataForm.voteDown,
                'level': this.dataForm.level,
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
