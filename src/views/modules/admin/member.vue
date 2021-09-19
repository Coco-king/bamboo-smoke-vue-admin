<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('admin:member:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >
          新增
        </el-button>
        <el-button
          v-if="isAuth('admin:member:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >
          批量删除
        </el-button>
      </el-form-item>
    </el-form>
    <el-table
      ref="filterTable"
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      ></el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="ID"
        prop="id"
        width="80"
      >
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="头像"
        width="60"
      >
        <template slot-scope="scope">
          <el-popover placement="left-start" title="" trigger="hover" >
            <img :src="scope.row.avatar" alt="" style="width: 150px;height: 150px">
            <img alt="" slot="reference" :src="scope.row.avatar" style="width: 30px;height: 30px">
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column
        prop="memberName"
        header-align="center"
        align="center"
        label="用户名"
      ></el-table-column>
      <el-table-column
        prop="email"
        header-align="center"
        align="center"
        label="邮箱"
      ></el-table-column>
      <el-table-column
        prop="mobile"
        header-align="center"
        align="center"
        label="手机电话"
        width="130"
      ></el-table-column>
      <el-table-column
        prop="authName"
        header-align="center"
        align="center"
        label="社区认证"
        width="110"
      ></el-table-column>
      <el-table-column
        prop="point"
        header-align="center"
        align="center"
        label="积分"
        width="90"
      ></el-table-column>
      <el-table-column
        prop="vipLevel"
        header-align="center"
        align="center"
        label="VIP等级"
        width="80"
      ></el-table-column>
      <el-table-column
        prop="gender"
        header-align="center"
        align="center"
        label="性别"
        width="60"
      >
        <template slot-scope="scope">
          <el-tag :type="scope.row.gender === '1' ? 'primary' : 'danger'">
            {{ scope.row.gender === '1' ? '男' : '女' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态"
        width="110"
      >
        <template slot-scope="scope">
          <el-tag :type="statusType(scope.row.status)">
            {{ statusName(scope.row.status) }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建日期"
        width="110"
      >
        <template slot-scope="scope">
          <el-popover
            placement="left-start"
            title="创建日期"
            width="180"
            trigger="hover"
            :content="scope.row.createTime"
          >
            <el-button slot="reference" type="text">{{ scope.row.createTime.substring(0, 11) }}</el-button>
          </el-popover>
        </template>
      </el-table-column>
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
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from './member-add-or-update'

export default {
  data() {
    return {
      dataForm: {
        key: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      options: [
        {
          value: '0',
          label: '未激活邮箱',
          type: 'warning'
        },
        {
          value: '1',
          label: '正常',
          type: 'success'
        },
        {
          value: '-1',
          label: '已封禁',
          type: 'danger'
        },
        {
          value: '-10',
          label: '已注销账户',
          type: 'info'
        }
      ]
    }
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList()
  },
  computed: {
    statusType() {
      return function (status) {
        return this.searchStatus(status).type
      }
    },
    statusName() {
      return function (status) {
        return this.searchStatus(status).label
      }
    },
    searchStatus() {
      return function (status) {
        return this.options.filter(item => item.value === status)[0]
      }
    }
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/admin/member/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list
          this.totalPage = data.page.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val
      this.getDataList()
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
          url: this.$http.adornUrl('/admin/member/delete'),
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
