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
          placeholder="输入标题/创建人"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <category-select v-model="dataForm.categoryId"></category-select>
      </el-form-item>
      <el-form-item>
        <el-select
          style="width: 150px;"
          v-model="dataForm.status"
          clearable
          placeholder="请选择文章状态"
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
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('admin:article:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >
          新增
        </el-button>
        <el-button
          v-if="isAuth('admin:article:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >
          批量删除
        </el-button>
        <el-popconfirm
          style="margin-left: 0.71em"
          title="是否将选择数据通过审核，已审核的不会被改变"
          confirm-button-text="是"
          cancel-button-text="否"
          cancel-button-type="default"
          @confirm="examine(null, 1)"
          @cancel="examine(null, -1)"
          icon="el-icon-info"
        >
          <el-button
            type="success"
            slot="reference"
            :disabled="dataListSelections.length <= 0"
          >
            批量审核
          </el-button>
        </el-popconfirm>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="社区认证">
              <span>{{ props.row.memberAuthName }}</span>
            </el-form-item>
            <el-form-item label="点赞量">
              <span>{{ props.row.voteUp }}</span>
            </el-form-item>
            <el-form-item label="点踩量">
              <span>{{ props.row.voteDown }}</span>
            </el-form-item>
            <el-form-item label="评论数量">
              <span>{{ props.row.commentCount }}</span>
            </el-form-item>
            <el-form-item label="置顶等级">
              <span>{{ props.row.level }}</span>
            </el-form-item>
            <el-form-item label="是否精华">
              <span>{{ props.row.recommend ? '是' : '否' }}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      ></el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="ID"
        width="120"
      ></el-table-column>
      <el-table-column
        prop="title"
        header-align="center"
        align="center"
        label="标题"
      >
        <template slot-scope="scope">
          <el-link :underline="false" @click="viewDetail(scope.row.id)">{{ scope.row.title }}</el-link>
        </template>
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="封面"
        width="90"
      >
        <template slot-scope="scope">
          <el-popover placement="left-start" title="" trigger="hover">
            <el-image
              style="width: 480px; height: 270px"
              :src="scope.row.coverImage || coverDefaultSrc"
              fit="contain"
            ></el-image>
            <el-image
              fit="contain"
              :src="scope.row.coverImage || coverDefaultSrc"
              style="width: 48px;height: 27px"
              slot="reference"
            ></el-image>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column
        prop="memberName"
        header-align="center"
        align="center"
        label="创建人"
        width="120"
      ></el-table-column>
      <el-table-column
        prop="categoryName"
        header-align="center"
        align="center"
        label="分类"
        width="90"
      ></el-table-column>
      <el-table-column
        prop="viewCount"
        header-align="center"
        align="center"
        label="访问量"
        width="110"
      ></el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态"
        width="120"
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
        width="160"
      ></el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="130"
        label="操作"
      >
        <template slot-scope="scope">
          <el-popconfirm
            v-if="scope.row.status === '0'"
            style="margin-right: 0.71em"
            title="是否通过审核"
            confirm-button-text="是"
            cancel-button-text="否"
            cancel-button-type="default"
            @confirm="examine(scope.row.id, 1)"
            @cancel="examine(scope.row.id, -1)"
            icon="el-icon-info"
          >
            <el-button slot="reference" size="small" type="text">审核</el-button>
          </el-popconfirm>
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
    <!-- 审核页 -->
    <article-view v-if="viewVisible" ref="articleView"></article-view>
  </div>
</template>

<script>
import AddOrUpdate from './article-add-or-update'
import ArticleView from './article-view'
import CategorySelect from '@/components/category-select'

export default {
  data() {
    return {
      coverDefaultSrc: 'https://cube.elemecdn.com/6/94/4d3ea53c084bad6931a56d5158a48jpeg.jpeg',
      dataForm: {
        key: '',
        categoryId: '',
        id: '',
        status: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      viewVisible: false,
      options: [
        {
          value: '0',
          label: '审核中',
          type: 'warning'
        },
        {
          value: '1',
          label: '审核通过',
          type: 'success'
        },
        {
          value: '-1',
          label: '审核不通过',
          type: 'danger'
        }
      ]
    }
  },
  components: {
    AddOrUpdate, ArticleView, CategorySelect
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
  activated() {
    this.getDataList()
  },
  methods: {
    // 预览文章
    viewDetail(id) {
      this.viewVisible = true
      this.$nextTick(() => {
        this.$refs.articleView.init(id)
      })
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/admin/article/list'),
        method: 'get',
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          categoryId: this.dataForm.categoryId,
          id: this.dataForm.id,
          status: this.dataForm.status
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
    // 审核
    examine(id, status, isSon = false) {
      let ids = id ? [id] : this.dataListSelections.map(item => item.id)
      this.$http({
        url: this.$http.adornUrl('/admin/article/examine'),
        method: 'put',
        data: this.$http.adornData({ids, status}, false)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1000,
            onClose: () => {
              if (isSon) {
                this.$refs.articleView.closeArticleView()
              }
              this.getDataList()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
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
          url: this.$http.adornUrl('/admin/article/delete'),
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

<style>
.demo-table-expand {
  font-size: 0;
}

.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}

.demo-table-expand .el-form-item {
  margin: 0 2%;
  width: 46%;
}
</style>
