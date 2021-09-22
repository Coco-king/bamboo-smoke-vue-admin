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
      <el-form-item label="分类名" prop="name">
        <el-input v-model="dataForm.name" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="内容描述" prop="content">
        <el-input v-model="dataForm.content" placeholder="内容描述"></el-input>
      </el-form-item>
      <el-form-item label="分类概要" prop="summary">
        <el-input v-model="dataForm.summary" placeholder="分类概要"></el-input>
      </el-form-item>
      <el-form-item label="图标" prop="icon">
        <el-popover
          ref="iconListPopover"
          placement="bottom-start"
          trigger="click"
          popper-class="mod-menu__icon-popover"
          v-model="iconVisible"
        >
          <div class="mod-menu__icon-inner">
            <div class="mod-menu__icon-list">
              <el-button
                v-for="(item, index) in iconList"
                :key="index"
                @click="iconActiveHandle(item)"
                :class="{ 'is-active': item === dataForm.icon }"
              >
                <icon-svg :name="item"></icon-svg>
              </el-button>
            </div>
            <div style="text-align: right;margin-right: 3.39em;margin-top: 0.6em">
              <el-button size="mini" type="text" @click="rollbackIcon">取消</el-button>
              <el-button type="primary" size="mini" @click="iconVisible = false">确定</el-button>
            </div>
          </div>
        </el-popover>
        <icon-svg v-if="dataForm.icon" :name="dataForm.icon" class="icon-svg-selected"></icon-svg>
        <el-button size="small" v-popover:iconListPopover>点击选择</el-button>
      </el-form-item>
      <el-form-item label="分类内容数量" prop="articleCount">
        <el-input
          v-model="dataForm.articleCount"
          placeholder="该分类的内容数量"
        ></el-input>
      </el-form-item>
      <el-form-item label="排序编码" prop="order">
        <el-input v-model="dataForm.order" placeholder="排序编码"></el-input>
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
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import Icon from '@/icons'

export default {
  data() {
    return {
      visible: false,
      iconVisible: false,
      iconList: [],
      initIcon: '',
      dataForm: {
        id: 0,
        name: '',
        content: '',
        summary: '',
        icon: '',
        articleCount: '',
        order: '',
        metaKeywords: '',
        metaDescription: '',
        status: ''
      },
      dataRule: {
        name: [{required: true, message: '标题不能为空', trigger: 'blur'}],
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
        status: [
          {required: true, message: '分类状态不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  created() {
    this.iconList = Icon.getNameList()
  },
  methods: {
    // 图标回滚到初始状态
    rollbackIcon() {
      this.dataForm.icon = this.initIcon
      this.iconVisible = false
    },
    // 图标选中
    iconActiveHandle(iconName) {
      if (!this.initIcon) {
        this.initIcon = this.dataForm.icon
      }
      this.dataForm.icon = iconName
    },
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
              this.dataForm.metaKeywords = data.category.metaKeywords
              this.dataForm.metaDescription = data.category.metaDescription
              this.dataForm.status = data.category.status
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
              metaKeywords: this.dataForm.metaKeywords,
              metaDescription: this.dataForm.metaDescription,
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

<style lang="scss">
.mod-menu {
  .menu-list__input,
  .icon-list__input {
    > .el-input__inner {
      cursor: pointer;
    }
  }

  &__icon-popover {
    width: 458px;
    overflow: hidden;
  }

  &__icon-inner {
    width: 478px;
    max-height: 258px;
    overflow-x: hidden;
    overflow-y: auto;
  }

  &__icon-list {
    width: 458px;
    padding: 0;
    margin: -8px 0 0 -8px;

    > .el-button {
      padding: 8px;
      margin: 8px 0 0 8px;

      > span {
        display: inline-block;
        vertical-align: middle;
        width: 18px;
        height: 18px;
        font-size: 18px;
      }
    }
  }

  .icon-list__tips {
    font-size: 18px;
    text-align: center;
    color: #e6a23c;
    cursor: pointer;
  }
}
</style>

<style scoped>
.icon-svg-selected {
  width: 1.6em;
  height: 1.6em;
  fill: currentColor;
  overflow: hidden;
}
</style>
