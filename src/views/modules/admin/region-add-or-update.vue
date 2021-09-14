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
      <el-form-item label="该地区的上级区域" prop="parentId">
        <el-input
          v-model="dataForm.parentId"
          placeholder="该地区的上级区域"
        ></el-input>
      </el-form-item>
      <el-form-item label="地区名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="地区名称"></el-input>
      </el-form-item>
      <el-form-item label="地区对应值" prop="value">
        <el-input v-model="dataForm.value" placeholder="地区对应值"></el-input>
      </el-form-item>
      <el-form-item label="区域层级" prop="level">
        <el-input v-model="dataForm.level" placeholder="区域层级"></el-input>
      </el-form-item>
      <el-form-item label="逻辑删除（0：未删除，1：已删除）" prop="deleted">
        <el-input
          v-model="dataForm.deleted"
          placeholder="逻辑删除（0：未删除，1：已删除）"
        ></el-input>
      </el-form-item>
      <el-form-item label="创建日期" prop="createTime">
        <el-input
          v-model="dataForm.createTime"
          placeholder="创建日期"
        ></el-input>
      </el-form-item>
      <el-form-item label="最后修改日期" prop="updateTime">
        <el-input
          v-model="dataForm.updateTime"
          placeholder="最后修改日期"
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
        parentId: '',
        name: '',
        value: '',
        level: '',
        deleted: '',
        createTime: '',
        updateTime: ''
      },
      dataRule: {
        parentId: [
          {
            required: true,
            message: '该地区的上级区域不能为空',
            trigger: 'blur'
          }
        ],
        name: [
          { required: true, message: '地区名称不能为空', trigger: 'blur' }
        ],
        value: [
          { required: true, message: '地区对应值不能为空', trigger: 'blur' }
        ],
        level: [
          { required: true, message: '区域层级不能为空', trigger: 'blur' }
        ],
        deleted: [
          {
            required: true,
            message: '逻辑删除（0：未删除，1：已删除）不能为空',
            trigger: 'blur'
          }
        ],
        createTime: [
          { required: true, message: '创建日期不能为空', trigger: 'blur' }
        ],
        updateTime: [
          { required: true, message: '最后修改日期不能为空', trigger: 'blur' }
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
            url: this.$http.adornUrl(`/admin/region/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.parentId = data.region.parentId
              this.dataForm.name = data.region.name
              this.dataForm.value = data.region.value
              this.dataForm.level = data.region.level
              this.dataForm.deleted = data.region.deleted
              this.dataForm.createTime = data.region.createTime
              this.dataForm.updateTime = data.region.updateTime
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
              `/admin/region/${!this.dataForm.id ? 'save' : 'update'}`
            ),
            method: `${!this.dataForm.id ? 'post' : 'put'}`,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              parentId: this.dataForm.parentId,
              name: this.dataForm.name,
              value: this.dataForm.value,
              level: this.dataForm.level,
              deleted: this.dataForm.deleted,
              createTime: this.dataForm.createTime,
              updateTime: this.dataForm.updateTime
            })
          }).then(({ data }) => {
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
