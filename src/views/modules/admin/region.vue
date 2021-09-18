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
      ref="table"
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
      <el-table-column prop="updateTime" label="最后修改时间"></el-table-column>
      <el-table-column fixed="right" label="操作" width="268">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.level <= 2 && isAuth('admin:region:save')"
            @click="addOrUpdateHandle(null, scope.row.id)"
            type="success"
            plain
            size="mini"
          >
            添加下级区域
          </el-button>
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
    refreshData(data) {
      const newParentId = data[0]
      const oldItem = data[1]
      // 懒加载刷新旧的父级
      if (oldItem) {
        this.deleteLazyTableItem(oldItem)
      }
      // 懒加载刷新新的父级
      if (newParentId) {
        this.fetchData(newParentId).then(({data}) => {
          if (data && data.code === 0) {
            this.$set(this.$refs.table.store.states.lazyTreeNodeMap, newParentId, data.list)
          }
        })
      }
    },
    load(tree, treeNode, resolve) {
      this.fetchData(tree.id).then(({data}) => {
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
      this.fetchData(0).then(({data}) => {
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
    addOrUpdateHandle(id, regionId) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id, regionId)
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
              this.$router.replace({path: '/refresh'})
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
                this.deleteLazyTableItem(row)
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
    deleteLazyTableItem(item) {
      const store = this.$refs.table.store
      // id等于1时表示为根节点
      if (item.id !== '1') {
        let parentRow = store.states.data.find(child => child.id === item.parentId)
        if (!parentRow) {
          const keys = Object.keys(store.states.lazyTreeNodeMap)
          for (let i = 0; i < keys.length; i++) {
            parentRow = store.states.lazyTreeNodeMap[keys[i]].find(child => child.id === item.parentId)
            if (parentRow) {
              break
            }
          }
        }
        const parent = store.states.lazyTreeNodeMap[item.parentId]
        const index = parent.findIndex(child => child.id === item.id)
        parent.splice(index, 1)
      } else {
        const parent = store.states.data
        const index = parent.findIndex(child => child.id === item.id)
        parent.splice(index, 1)
      }
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
