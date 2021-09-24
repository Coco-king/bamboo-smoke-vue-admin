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
          v-model="dataForm.id"
          prefix-icon="el-icon-search"
          placeholder="输入ID"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          prefix-icon="el-icon-search"
          placeholder="输入邮箱/会员名/手机号"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-select
          style="width: 120px;"
          v-model="dataForm.gender"
          clearable
          placeholder="请选择性别"
        >
          <el-option
            v-for="item in [{label:'男',value:'1'},{label:'女',value:'0'}]"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-select
          style="width: 150px;"
          v-model="dataForm.status"
          clearable
          placeholder="请选择会员状态"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-date-picker
          v-model="dataForm.createTime"
          type="daterange"
          align="left"
          unlink-panels
          value-format="yyyy-MM-dd"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :picker-options="pickerOptions"
        >
        </el-date-picker>
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
        <el-popover
          placement="top"
          trigger="hover"
          style="margin-left: 0.71em"
        >
          <el-button
            plain
            type="success"
            @click="status(null, 1, true)"
            :disabled="dataListSelections.length <= 0"
          >
            批量激活
          </el-button>
          <el-button
            plain
            type="primary"
            @click="status(null, 1)"
            :disabled="dataListSelections.length <= 0"
          >
            批量解封
          </el-button>
          <el-button
            type="success"
            @click="status(null, 1, false, true)"
            :disabled="dataListSelections.length <= 0"
          >
            批量恢复
          </el-button>
          <el-button
            plain
            type="warning"
            @click="status(null, -1)"
            :disabled="dataListSelections.length <= 0"
          >
            批量封禁
          </el-button>
          <el-button
            plain
            type="danger"
            @click="status(null, -10)"
            :disabled="dataListSelections.length <= 0"
          >
            批量销户
          </el-button>
          <el-button
            v-if="isAuth('admin:member:delete')"
            type="danger"
            @click="deleteHandle()"
            :disabled="dataListSelections.length <= 0"
          >
            批量删除
          </el-button>
          <el-button slot="reference" type="info" :disabled="dataListSelections.length <= 0">批量操作</el-button>
        </el-popover>
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
        sortable
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
          <el-popover placement="left-start" title="" trigger="hover">
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
        sortable
        prop="point"
        header-align="center"
        align="center"
        label="积分"
        width="90"
      ></el-table-column>
      <el-table-column
        sortable
        prop="vipLevel"
        header-align="center"
        align="center"
        label="VIP"
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
        label="注册日期"
        width="110"
      >
        <template slot-scope="scope">
          <el-popover
            placement="left-start"
            title="详细注册日期"
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
        width="170"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.status !== '-10'"
            type="text"
            size="small"
            @click="status(scope.row.id, -10)"
          >
            销户
          </el-button>
          <el-button
            v-if="scope.row.status === '-10'"
            type="text"
            size="small"
            @click="status(scope.row.id, 1, false, true)"
          >
            恢复
          </el-button>
          <el-button
            v-if="scope.row.status === '-1'"
            type="text"
            size="small"
            @click="status(scope.row.id, 1)"
          >
            解封
          </el-button>
          <el-button
            v-if="scope.row.status === '1'"
            type="text"
            size="small"
            @click="status(scope.row.id, -1)"
          >
            封禁
          </el-button>
          <el-button
            v-if="scope.row.status === '0'"
            type="text"
            size="small"
            @click="status(scope.row.id, 1, true)"
          >
            激活
          </el-button>
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
        id: '',
        key: '',
        gender: '',
        status: '',
        createTime: ''
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
      ],
      pickerOptions: {
        shortcuts: [{
          text: '最近一周',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '最近一个月',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '最近三个月',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      }
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
    // 修改状态
    status(id, status, isActive = false, isRecovery = false) {
      let ids = id ? [id] : this.dataListSelections.map(item => item.id)
      this.$http({
        url: this.$http.adornUrl('/admin/member/status'),
        method: 'put',
        data: this.$http.adornData({ids, status, isActive, isRecovery}, false)
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
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/admin/member/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          id: this.dataForm.id,
          key: this.dataForm.key,
          gender: this.dataForm.gender,
          status: this.dataForm.status,
          createBeginTime: this.dataForm.createTime[0],
          createEndTime: this.dataForm.createTime[1]
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
