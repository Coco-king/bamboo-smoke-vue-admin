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
      <el-row>
        <el-col :span="9">&nbsp;</el-col>
        <el-col :span="4">
          <el-form-item prop="avatar">
            <mini-upload
              ref="miniUpload"
              class="avatar-uploader"
              type="avatar"
              @change="handleAvatarChange"
            />
          </el-form-item>
        </el-col>
        <el-col :span="11">&nbsp;</el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="用户名" prop="memberName">
            <el-input
              v-model="dataForm.memberName"
              placeholder="昵称"
            ></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="微信号" prop="wechat">
            <el-input v-model="dataForm.wechat" placeholder="微信号"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="密码" prop="password">
            <el-input v-model="dataForm.password" type="password" show-password placeholder="密码"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="确认密码" prop="rePass">
            <el-input type="password" v-model="dataForm.rePass" show-password placeholder="确认密码"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="手机号码" prop="mobile">
            <el-input
              v-model="dataForm.mobile"
              placeholder="手机号码"
            ></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="社区认证" prop="authName">
            <el-input
              v-model="dataForm.authName"
              placeholder="社区认证"
            ></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="生日" prop="birthday">
            <el-date-picker
              type="date"
              value-format="yyyy-MM-dd"
              placeholder="生日"
              v-model="dataForm.birthday"
              style="width: 100%;"
            ></el-date-picker>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="积分" prop="point">
            <el-input v-model="dataForm.point" placeholder="积分"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="vip等级" prop="vipLevel">
            <el-input
              v-model="dataForm.vipLevel"
              placeholder="vip等级"
            ></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="性别" prop="gender">
            <el-radio-group v-model="dataForm.gender">
              <el-radio label="1" border>男</el-radio>
              <el-radio label="0" border>女</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
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
      </el-row>
      <el-row>
        <el-col :span="23">
          <el-form-item label="所在城市" prop="city">
            <region-cascader ref="cascader" @change="handleChange" :maxLevel="3"/>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="23">
          <el-form-item label="个性签名" prop="sign">
            <el-input
              type="textarea"
              v-model="dataForm.sign"
              :autosize="{ minRows: 2, maxRows: 4 }"
              placeholder="请输入个性签名"
            ></el-input>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { isEmail, isMobile } from '@/utils/validate'
import regionCascader from '@/components/region-cascader'
import miniUpload from '@/components/upload/miniUpload'

export default {
  components: {regionCascader, miniUpload},
  data() {
    let checkEmail = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('邮箱不能为空'))
      }
      if (!isEmail(value)) {
        callback(new Error('邮箱格式不正确'))
      } else {
        callback()
      }
    }

    let checkPoint = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('积分不能为空'))
      }
      if (!Number.isInteger(Number.parseInt(value))) {
        callback(new Error('请输入一个数字'))
      } else {
        if (value < 0) {
          callback(new Error('必须是0或者以上的数字'))
        } else {
          callback()
        }
      }
    }

    let checkRePass = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.dataForm.password) {
        callback(new Error('两次输入密码不一致'))
      } else {
        callback()
      }
    }

    let checkPhone = (rule, value, callback) => {
      if (!value) {
        callback(new Error('电话号码不能为空'))
      } else if (!isMobile(value)) {
        callback(new Error('电话号码格式有误'))
      } else {
        callback()
      }
    }

    return {
      visible: false,
      dataForm: {
        id: 0,
        memberName: '',
        password: '',
        rePass: '',
        authName: '',
        email: '',
        mobile: '',
        city: '',
        sign: '',
        wechat: '',
        birthday: '',
        avatar: '',
        vipLevel: '0',
        gender: '1',
        point: '0',
        status: '0'
      },
      dataRule: {
        avatar: [{required: true, message: '头像不能为空', trigger: 'blur'}],
        memberName: [
          {required: true, message: '昵称不能为空', trigger: 'blur'}
        ],
        password: [
          {required: true, message: '密码不能为空', trigger: 'blur'}
        ],
        rePass: [{required: true, validator: checkRePass, trigger: 'blur'}],
        email: [{required: true, validator: checkEmail, trigger: 'blur'}],
        mobile: [{required: true, validator: checkPhone, trigger: 'blur'}],
        point: [{required: true, validator: checkPoint, trigger: 'blur'}],
        gender: [{required: true, message: '性别不能为空', trigger: 'blur'}],
        vipLevel: [
          {required: true, message: 'vip等级不能为空', trigger: 'blur'}
        ],
        status: [{required: true, message: '状态不能为空', trigger: 'blur'}]
      },
      options: [
        {
          value: '0',
          label: '未激活邮箱'
        },
        {
          value: '1',
          label: '正常'
        },
        {
          value: '-1',
          label: '已封禁'
        },
        {
          value: '-10',
          label: '已注销账户'
        }
      ]
    }
  },
  methods: {
    handleChange(value) {
      this.dataForm.city = value[value.length - 1]
    },
    handleAvatarChange(value) {
      this.dataForm.avatar = value
    },
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()

        const $miniUpload = this.$refs['miniUpload']
        $miniUpload.resetFields()

        const $cascader = this.$refs['cascader']
        $cascader.resetFields()

        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/member/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              if (data.member.city) {
                $cascader.initSelect(data.member.city, false)
              }

              if (data.member.avatar) {
                $miniUpload.init(data.member.avatar)
              }
              this.dataForm.memberName = data.member.memberName
              this.dataForm.authName = data.member.authName
              this.dataForm.password = data.member.password
              this.dataForm.rePass = data.member.password
              this.dataForm.email = data.member.email
              this.dataForm.mobile = data.member.mobile
              this.dataForm.city = data.member.city
              this.dataForm.point = data.member.point
              this.dataForm.sign = data.member.sign
              this.dataForm.gender = data.member.gender
              this.dataForm.wechat = data.member.wechat
              this.dataForm.vipLevel = data.member.vipLevel
              this.dataForm.birthday = data.member.birthday
              this.dataForm.avatar = data.member.avatar
              this.dataForm.status = data.member.status
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
              `/admin/member/${!this.dataForm.id ? 'save' : 'update'}`
            ),
            method: `${!this.dataForm.id ? 'post' : 'put'}`,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              memberName: this.dataForm.memberName,
              password: this.dataForm.password,
              rePass: this.dataForm.rePass,
              authName: this.dataForm.authName,
              email: this.dataForm.email,
              mobile: this.dataForm.mobile,
              city: this.dataForm.city,
              point: this.dataForm.point,
              sign: this.dataForm.sign,
              gender: this.dataForm.gender,
              wechat: this.dataForm.wechat,
              vipLevel: this.dataForm.vipLevel,
              birthday: this.dataForm.birthday,
              avatar: this.dataForm.avatar,
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

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}

.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 80px;
  height: 80px;
  line-height: 80px;
  border-radius: 20px;
  text-align: center;
}

.avatar {
  width: 80px;
  height: 80px;
  border-radius: 20px;
  display: block;
}

.el-dialog__body {
  padding-top: 5px;
}
</style>
