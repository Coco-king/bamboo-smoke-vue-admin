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
      label-width="120px"
    >
      <el-form-item label="上级区域" prop="parentId">
        <el-cascader
          style="width:100%"
          v-model="cascadeData"
          :options="regionList"
          :props="props"
          @change="handleChange"
          change-on-select
          clearable
        ></el-cascader>
      </el-form-item>
      <el-form-item label="地区名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="地区名称"></el-input>
      </el-form-item>
      <el-form-item label="地区对应值" prop="value">
        <el-input
          v-model="dataForm.value"
          :disabled="!!this.dataForm.id"
          placeholder="地区对应值"
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
      regionList: [],
      cascadeData: [],
      props: {
        checkStrictly: true,
        label: 'name',
        children: 'children',
        expandTrigger: 'hover'
      },
      dataForm: {
        id: 0,
        parentId: '',
        name: '',
        value: ''
      },
      dataRule: {
        parentId: [
          {
            required: true,
            message: '该地区的上级区域不能为空',
            trigger: 'blur'
          }
        ],
        name: [{required: true, message: '地区名称不能为空', trigger: 'blur'}]
      }
    }
  },
  methods: {
    handleChange(value) {
      this.dataForm.parentId = value[value.length - 1]
    },
    init(id) {
      this.dataForm.id = id || 0

      this.$http({
        url: this.$http.adornUrl('/admin/region/list/tree'),
        method: 'get',
        params: this.$http.adornParams({maxLevel: 2})
      })
      .then(({data}) => {
        if (data && data.code === 0) {
          this.regionList = data.list
        }
      })
      .then(() => {
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.cascadeData = []
        })
      })
      .then(() => {
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/region/info/${this.dataForm.id}`
            ),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.cascadeData = data.cascadeData
              this.dataForm.id = data.region.id
              this.dataForm.parentId = data.region.parentId
              this.dataForm.name = data.region.name
              this.dataForm.value = data.region.value
              this.dataForm.level = data.region.level
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
              level: this.dataForm.level,
              value: this.dataForm.value
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1000,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList', [data.newParentId, data.oldItem])
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
