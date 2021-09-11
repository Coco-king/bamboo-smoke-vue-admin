<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="用户ID" prop="memberId">
      <el-input v-model="dataForm.memberId" placeholder="用户ID"></el-input>
    </el-form-item>
    <el-form-item label="社交用户id" prop="socialUid">
      <el-input v-model="dataForm.socialUid" placeholder="社交用户id"></el-input>
    </el-form-item>
    <el-form-item label="社交平台名称" prop="socialName">
      <el-input v-model="dataForm.socialName" placeholder="社交平台名称"></el-input>
    </el-form-item>
    <el-form-item label="社交平台类型" prop="socialType">
      <el-input v-model="dataForm.socialType" placeholder="社交平台类型"></el-input>
    </el-form-item>
    <el-form-item label="用户此次登录的Access Token" prop="accessToken">
      <el-input v-model="dataForm.accessToken" placeholder="用户此次登录的Access Token"></el-input>
    </el-form-item>
    <el-form-item label="针对QQ，在授权自动续期步骤中，获取新的Access_Token时需要提供的参数" prop="refreshToken">
      <el-input v-model="dataForm.refreshToken" placeholder="针对QQ，在授权自动续期步骤中，获取新的Access_Token时需要提供的参数"></el-input>
    </el-form-item>
    <el-form-item label="Access Token过期时间" prop="expiresIn">
      <el-input v-model="dataForm.expiresIn" placeholder="Access Token过期时间"></el-input>
    </el-form-item>
    <el-form-item label="绑定状态" prop="status">
      <el-input v-model="dataForm.status" placeholder="绑定状态"></el-input>
    </el-form-item>
    <el-form-item label="创建日期" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="创建日期"></el-input>
    </el-form-item>
    <el-form-item label="修改日期" prop="updateTime">
      <el-input v-model="dataForm.updateTime" placeholder="修改日期"></el-input>
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
          socialUid: '',
          socialName: '',
          socialType: '',
          accessToken: '',
          refreshToken: '',
          expiresIn: '',
          status: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          memberId: [
            { required: true, message: '用户ID不能为空', trigger: 'blur' }
          ],
          socialUid: [
            { required: true, message: '社交用户id不能为空', trigger: 'blur' }
          ],
          socialName: [
            { required: true, message: '社交平台名称不能为空', trigger: 'blur' }
          ],
          socialType: [
            { required: true, message: '社交平台类型不能为空', trigger: 'blur' }
          ],
          accessToken: [
            { required: true, message: '用户此次登录的Access Token不能为空', trigger: 'blur' }
          ],
          refreshToken: [
            { required: true, message: '针对QQ，在授权自动续期步骤中，获取新的Access_Token时需要提供的参数不能为空', trigger: 'blur' }
          ],
          expiresIn: [
            { required: true, message: 'Access Token过期时间不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '绑定状态不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '创建日期不能为空', trigger: 'blur' }
          ],
          updateTime: [
            { required: true, message: '修改日期不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/api/membersocial/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.memberId = data.memberSocial.memberId
                this.dataForm.socialUid = data.memberSocial.socialUid
                this.dataForm.socialName = data.memberSocial.socialName
                this.dataForm.socialType = data.memberSocial.socialType
                this.dataForm.accessToken = data.memberSocial.accessToken
                this.dataForm.refreshToken = data.memberSocial.refreshToken
                this.dataForm.expiresIn = data.memberSocial.expiresIn
                this.dataForm.status = data.memberSocial.status
                this.dataForm.createTime = data.memberSocial.createTime
                this.dataForm.updateTime = data.memberSocial.updateTime
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
              url: this.$http.adornUrl(`/api/membersocial/${!this.dataForm.id ? 'save' : 'update'}`),
              method: `${!this.dataForm.id ? 'post' : 'put'}`,
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'memberId': this.dataForm.memberId,
                'socialUid': this.dataForm.socialUid,
                'socialName': this.dataForm.socialName,
                'socialType': this.dataForm.socialType,
                'accessToken': this.dataForm.accessToken,
                'refreshToken': this.dataForm.refreshToken,
                'expiresIn': this.dataForm.expiresIn,
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
