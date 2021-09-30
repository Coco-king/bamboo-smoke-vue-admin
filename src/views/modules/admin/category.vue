<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          style="width: 150px;"
          prefix-icon="el-icon-search"
          v-model="dataForm.id"
          placeholder="输入ID"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-input
          prefix-icon="el-icon-search"
          v-model="dataForm.key"
          placeholder="输入分类名/SEO关键字"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('admin:category:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >
          新增
        </el-button>
        <el-button
          v-if="isAuth('admin:category:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >
          批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      style="width: 100%;margin-bottom: 20px;"
      border
      stripe
      row-key="id"
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      :tree-props="{ children: 'children' }"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      ></el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="分类名"
      ></el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="分类ID"
      ></el-table-column>
      <el-table-column
        prop="content"
        header-align="center"
        align="center"
        label="内容描述"
      ></el-table-column>
      <el-table-column
        prop="summary"
        header-align="center"
        align="center"
        label="分类概要"
      ></el-table-column>
      <el-table-column
        prop="icon"
        header-align="center"
        align="center"
        label="图标"
      >
        <template slot-scope="scope">
          <icon-svg :name="scope.row.icon"></icon-svg>
        </template>
      </el-table-column>
      <el-table-column
        prop="articleCount"
        header-align="center"
        align="center"
        label="分类内容数量"
      ></el-table-column>
      <el-table-column
        prop="order"
        header-align="center"
        align="center"
        label="排序编码"
      ></el-table-column>
      <el-table-column
        prop="metaKeywords"
        header-align="center"
        align="center"
        label="SEO关键字"
      ></el-table-column>
      <el-table-column
        prop="metaDescription"
        header-align="center"
        align="center"
        label="SEO描述内容"
      ></el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="分类状态"
      ></el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建日期"
        width="160"
      ></el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="100"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
          >
            修改
          </el-button>
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './category-add-or-update'
import { treeDataTranslate } from '@/utils'

export default {
  data() {
    return {
      dataForm: {
        id: '',
        key: ''
      },
      dataList: [],
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/api/category/list'),
        method: 'get',
        params: this.$http.adornParams({
          key: this.dataForm.key,
          id: this.dataForm.id
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = treeDataTranslate(data.list)
        } else {
          this.dataList = []
        }
        this.dataListLoading = false
      })
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle(id) {
      let ids = id ? [id] : this.dataListSelections.map(item => item.id)
      this.$confirm(
        `确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl('/admin/category/delete'),
          method: 'delete',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    }
  }
}
</script>

<style scoped>
.icon-svg {
  width: 1.6em;
  height: 1.6em;
  fill: currentColor;
  overflow: hidden;
}
</style>
