<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="标题" prop="title">
      <el-input v-model="dataForm.title" placeholder="标题"></el-input>
    </el-form-item>
    <el-form-item label="标题" prop="coverImage">
      <el-input v-model="dataForm.coverImage" placeholder="标题"></el-input>
    </el-form-item>
    <el-form-item label="内容" prop="content">
      <el-input v-model="dataForm.content" placeholder="内容"></el-input>
    </el-form-item>
    <el-form-item label="编辑模式：html可视化，markdown .." prop="editMode">
      <el-input v-model="dataForm.editMode" placeholder="编辑模式：html可视化，markdown .."></el-input>
    </el-form-item>
    <el-form-item label="文章所属分类ID" prop="categoryId">
      <el-input v-model="dataForm.categoryId" placeholder="文章所属分类ID"></el-input>
    </el-form-item>
    <el-form-item label="用户ID" prop="userId">
      <el-input v-model="dataForm.userId" placeholder="用户ID"></el-input>
    </el-form-item>
    <el-form-item label="用户认证标识" prop="userAuthName">
      <el-input v-model="dataForm.userAuthName" placeholder="用户认证标识"></el-input>
    </el-form-item>
    <el-form-item label="点赞人数" prop="voteUp">
      <el-input v-model="dataForm.voteUp" placeholder="点赞人数"></el-input>
    </el-form-item>
    <el-form-item label="点踩人数" prop="voteDown">
      <el-input v-model="dataForm.voteDown" placeholder="点踩人数"></el-input>
    </el-form-item>
    <el-form-item label="访问量" prop="viewCount">
      <el-input v-model="dataForm.viewCount" placeholder="访问量"></el-input>
    </el-form-item>
    <el-form-item label="评论数量" prop="commentCount">
      <el-input v-model="dataForm.commentCount" placeholder="评论数量"></el-input>
    </el-form-item>
    <el-form-item label="是否为精华" prop="recommend">
      <el-input v-model="dataForm.recommend" placeholder="是否为精华"></el-input>
    </el-form-item>
    <el-form-item label="置顶等级" prop="level">
      <el-input v-model="dataForm.level" placeholder="置顶等级"></el-input>
    </el-form-item>
    <el-form-item label="状态（0：审核未通过 1：审核通过）" prop="status">
      <el-input v-model="dataForm.status" placeholder="状态（0：审核未通过 1：审核通过）"></el-input>
    </el-form-item>
    <el-form-item label="逻辑删除（0：未删除，1：已删除）" prop="isDeleted">
      <el-input v-model="dataForm.isDeleted" placeholder="逻辑删除（0：未删除，1：已删除）"></el-input>
    </el-form-item>
    <el-form-item label="创建日期" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建日期"></el-input>
    </el-form-item>
    <el-form-item label="最后更新日期" prop="updateTime">
      <el-input v-model="dataForm.updateTime" placeholder="最后更新日期"></el-input>
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
          title: '',
          coverImage: '',
          content: '',
          editMode: '',
          categoryId: '',
          userId: '',
          userAuthName: '',
          voteUp: '',
          voteDown: '',
          viewCount: '',
          commentCount: '',
          recommend: '',
          level: '',
          status: '',
          isDeleted: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          title: [
            { required: true, message: '标题不能为空', trigger: 'blur' }
          ],
          coverImage: [
            { required: true, message: '标题不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '内容不能为空', trigger: 'blur' }
          ],
          editMode: [
            { required: true, message: '编辑模式：html可视化，markdown ..不能为空', trigger: 'blur' }
          ],
          categoryId: [
            { required: true, message: '文章所属分类ID不能为空', trigger: 'blur' }
          ],
          userId: [
            { required: true, message: '用户ID不能为空', trigger: 'blur' }
          ],
          userAuthName: [
            { required: true, message: '用户认证标识不能为空', trigger: 'blur' }
          ],
          voteUp: [
            { required: true, message: '点赞人数不能为空', trigger: 'blur' }
          ],
          voteDown: [
            { required: true, message: '点踩人数不能为空', trigger: 'blur' }
          ],
          viewCount: [
            { required: true, message: '访问量不能为空', trigger: 'blur' }
          ],
          commentCount: [
            { required: true, message: '评论数量不能为空', trigger: 'blur' }
          ],
          recommend: [
            { required: true, message: '是否为精华不能为空', trigger: 'blur' }
          ],
          level: [
            { required: true, message: '置顶等级不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '状态（0：审核未通过 1：审核通过）不能为空', trigger: 'blur' }
          ],
          isDeleted: [
            { required: true, message: '逻辑删除（0：未删除，1：已删除）不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建日期不能为空', trigger: 'blur' }
          ],
          updateTime: [
            { required: true, message: '最后更新日期不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/api/post/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.title = data.post.title
                this.dataForm.coverImage = data.post.coverImage
                this.dataForm.content = data.post.content
                this.dataForm.editMode = data.post.editMode
                this.dataForm.categoryId = data.post.categoryId
                this.dataForm.userId = data.post.userId
                this.dataForm.userAuthName = data.post.userAuthName
                this.dataForm.voteUp = data.post.voteUp
                this.dataForm.voteDown = data.post.voteDown
                this.dataForm.viewCount = data.post.viewCount
                this.dataForm.commentCount = data.post.commentCount
                this.dataForm.recommend = data.post.recommend
                this.dataForm.level = data.post.level
                this.dataForm.status = data.post.status
                this.dataForm.isDeleted = data.post.isDeleted
                this.dataForm.createTime = data.post.createTime
                this.dataForm.updateTime = data.post.updateTime
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
              url: this.$http.adornUrl(`/api/post/${!this.dataForm.id ? 'save' : 'update'}`),
              method: `${!this.dataForm.brandId ? 'post' : 'put'}`,
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'coverImage': this.dataForm.coverImage,
                'content': this.dataForm.content,
                'editMode': this.dataForm.editMode,
                'categoryId': this.dataForm.categoryId,
                'userId': this.dataForm.userId,
                'userAuthName': this.dataForm.userAuthName,
                'voteUp': this.dataForm.voteUp,
                'voteDown': this.dataForm.voteDown,
                'viewCount': this.dataForm.viewCount,
                'commentCount': this.dataForm.commentCount,
                'recommend': this.dataForm.recommend,
                'level': this.dataForm.level,
                'status': this.dataForm.status,
                'isDeleted': this.dataForm.isDeleted,
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
