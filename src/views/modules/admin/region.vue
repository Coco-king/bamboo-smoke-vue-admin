<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-button @click="getDataList()">刷新</el-button>
        <el-button
          v-if="isAuth('admin:region:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >
          新增
        </el-button>
        <el-button
          v-if="isAuth('admin:region:init')"
          type="success"
          @click="initRegionHandle()"
        >
          初始化
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      style="width: 100%;margin-bottom: 20px;"
      row-key="id"
      border
      stripe
      lazy
      :load="load"
      :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
    >
      <el-table-column prop="name" label="地区" sortable></el-table-column>
      <el-table-column prop="value" label="地区编号" sortable></el-table-column>
      <el-table-column prop="level" label="地区层级"></el-table-column>
      <el-table-column prop="createTime" label="创建时间"></el-table-column>
      <el-table-column fixed="right" label="操作" width="150">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.level !== 0"
            @click="addOrUpdateHandle(scope.row.id)"
            type="primary"
            plain
            size="mini"
          >
            修改
          </el-button>
          <el-button type="danger" plain size="mini" @click="deleteHandle(scope.row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="refreshData"
    ></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './region-add-or-update'

export default {
  data() {
    return {
      dataForm: {
        key: ''
      },
      dataList: [],
      dataListLoading: false,
      addOrUpdateVisible: false,
      maps: new Map()
    }
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList()
  },
  methods: {
    refreshData(data) {
      const newParentId = data[0]
      const oldParentId = data[1]
      // 懒加载刷新旧的父级
      if (oldParentId && this.maps.get(oldParentId)) {
        const {tree, treeNode, resolve} = this.maps.get(oldParentId)
        this.load(tree, treeNode, resolve)
      }
      // 懒加载刷新新的父级
      if (newParentId && this.maps.get(newParentId)) {
        const {tree, treeNode, resolve} = this.maps.get(newParentId)
        this.load(tree, treeNode, resolve)
      }
    },
    load(tree, treeNode, resolve) {
      const id = tree.id
      this.maps.set(id, {tree, treeNode, resolve})
      this.fetchData(id).then(({data}) => {
        if (data && data.code === 0) {
          resolve(data.list)
        } else {
          resolve([])
        }
      })
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.fetchData(1).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.list
        } else {
          this.dataList = []
        }
        this.dataListLoading = false
      })
    },
    // 拉取数据
    fetchData(rootId) {
      return this.$http({
        url: this.$http.adornUrl('/admin/region/list/tree'),
        method: 'get',
        params: this.$http.adornParams({parentId: rootId, isLazy: true})
      })
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    initRegionHandle() {
      this.$http({
        url: this.$http.adornUrl('/admin/region/initRegion'),
        method: 'post'
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '初始化成功',
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
    },
    // 删除
    deleteHandle(row) {
      this.$confirm(
        `确定对【${row.name}】${row.hasChildren ? '以及他的下级区域' : ''}进行删除操作?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl(`/admin/region/delete/${row.id}`),
          method: 'delete',
          data: this.$http.adornData({}, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                row.id === '1' ? this.getDataList() : this.refreshData([row.parentId])
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

<style>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
