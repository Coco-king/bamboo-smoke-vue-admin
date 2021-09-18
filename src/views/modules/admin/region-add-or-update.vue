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
        <region-cascader ref="cascader" @change="handleChange" :maxLevel="2"/>
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
import regionCascader from '@/components/region-cascader'

export default {
  components: {regionCascader},
  data() {
    let checkValue = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('地区对应值不能为空'))
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

    return {
      visible: false,
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
        name: [{required: true, message: '地区名称不能为空', trigger: 'blur'}],
        value: [{required: true, validator: checkValue, trigger: 'blur'}]
      }
    }
  },
  methods: {
    handleChange(value) {
      this.dataForm.parentId = value[value.length - 1]
    },
    init(id, regionId) {
      this.dataForm.id = id || 0

      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        const $cascader = this.$refs['cascader']
        $cascader.resetFields()

        if (regionId) {
          $cascader.initSelect(regionId, false)
        }

        if (this.dataForm.id) {
          $cascader.initSelect(this.dataForm.id)
          this.$http({
            url: this.$http.adornUrl(
              `/admin/region/info/${this.dataForm.id}`
            ),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
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
