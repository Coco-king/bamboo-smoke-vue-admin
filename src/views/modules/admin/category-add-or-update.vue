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
      <el-form-item label="标题" prop="name">
        <el-input v-model="dataForm.name" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="内容描述" prop="content">
        <el-input v-model="dataForm.content" placeholder="内容描述"></el-input>
      </el-form-item>
      <el-form-item label="分类概要" prop="summary">
        <el-input v-model="dataForm.summary" placeholder="分类概要"></el-input>
      </el-form-item>
      <el-form-item label="图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="图标"></el-input>
      </el-form-item>
      <el-form-item label="该分类的内容数量" prop="articleCount">
        <el-input
          v-model="dataForm.articleCount"
          placeholder="该分类的内容数量"
        ></el-input>
      </el-form-item>
      <el-form-item label="排序编码" prop="order">
        <el-input v-model="dataForm.order" placeholder="排序编码"></el-input>
      </el-form-item>
      <el-form-item label="父级分类的ID" prop="parentId">
        <el-input
          v-model="dataForm.parentId"
          placeholder="父级分类的ID"
        ></el-input>
      </el-form-item>
      <el-form-item label="SEO关键字" prop="metaKeywords">
        <el-input
          v-model="dataForm.metaKeywords"
          placeholder="SEO关键字"
        ></el-input>
      </el-form-item>
      <el-form-item label="SEO描述内容" prop="metaDescription">
        <el-input
          v-model="dataForm.metaDescription"
          placeholder="SEO描述内容"
        ></el-input>
      </el-form-item>
      <el-form-item label="分类状态" prop="status">
        <el-input v-model="dataForm.status" placeholder="分类状态"></el-input>
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
      <el-form-item label="修改日期" prop="updateTime">
        <el-input
          v-model="dataForm.updateTime"
          placeholder="修改日期"
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
        name: '',
        content: '',
        summary: '',
        icon: '',
        articleCount: '',
        order: '',
        parentId: '',
        metaKeywords: '',
        metaDescription: '',
        status: '',
        deleted: '',
        createTime: '',
        updateTime: ''
      },
      dataRule: {
        name: [{required: true, message: '标题不能为空', trigger: 'blur'}],
        content: [
          {required: true, message: '内容描述不能为空', trigger: 'blur'}
        ],
        summary: [
          {required: true, message: '分类概要不能为空', trigger: 'blur'}
        ],
        icon: [{required: true, message: '图标不能为空', trigger: 'blur'}],
        articleCount: [
          {
            required: true,
            message: '该分类的内容数量不能为空',
            trigger: 'blur'
          }
        ],
        order: [
          {required: true, message: '排序编码不能为空', trigger: 'blur'}
        ],
        parentId: [
          {required: true, message: '父级分类的ID不能为空', trigger: 'blur'}
        ],
        metaKeywords: [
          {required: true, message: 'SEO关键字不能为空', trigger: 'blur'}
        ],
        metaDescription: [
          {required: true, message: 'SEO描述内容不能为空', trigger: 'blur'}
        ],
        status: [
          {required: true, message: '分类状态不能为空', trigger: 'blur'}
        ],
        deleted: [
          {
            required: true,
            message: '逻辑删除（0：未删除，1：已删除）不能为空',
            trigger: 'blur'
          }
        ],
        createTime: [
          {required: true, message: '创建日期不能为空', trigger: 'blur'}
        ],
        updateTime: [
          {required: true, message: '修改日期不能为空', trigger: 'blur'}
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
            url: this.$http.adornUrl(
              `/admin/category/info/${this.dataForm.id}`
            ),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.category.name
              this.dataForm.content = data.category.content
              this.dataForm.summary = data.category.summary
              this.dataForm.icon = data.category.icon
              this.dataForm.articleCount = data.category.articleCount
              this.dataForm.order = data.category.order
              this.dataForm.parentId = data.category.parentId
              this.dataForm.metaKeywords = data.category.metaKeywords
              this.dataForm.metaDescription = data.category.metaDescription
              this.dataForm.status = data.category.status
              this.dataForm.deleted = data.category.deleted
              this.dataForm.createTime = data.category.createTime
              this.dataForm.updateTime = data.category.updateTime
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
              `/admin/category/${!this.dataForm.id ? 'save' : 'update'}`
            ),
            method: `${!this.dataForm.id ? 'post' : 'put'}`,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              content: this.dataForm.content,
              summary: this.dataForm.summary,
              icon: this.dataForm.icon,
              articleCount: this.dataForm.articleCount,
              order: this.dataForm.order,
              parentId: this.dataForm.parentId,
              metaKeywords: this.dataForm.metaKeywords,
              metaDescription: this.dataForm.metaDescription,
              status: this.dataForm.status,
              deleted: this.dataForm.deleted,
              createTime: this.dataForm.createTime,
              updateTime: this.dataForm.updateTime
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
