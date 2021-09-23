<template>
  <el-dialog
    width="60%"
    title="文章详情"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      disabled
      :model="dataForm"
      ref="dataForm"
      label-width="80px"
    >
      <el-row>
        <el-col :span="23">
          <el-form-item label="标题" prop="title">
            <el-input v-model="dataForm.title" placeholder="标题"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="23">
          <el-form-item label="封面" prop="coverImage">
            <img
              alt="封面"
              style="object-fit: contain;width: 480px;height: 270px"
              v-if="dataForm.coverImage"
              :src="dataForm.coverImage"
              class="avatar"
            >
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="分类" prop="categoryId">
            <el-input v-model="dataForm.categoryName" placeholder="分类"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="编辑模式" prop="editMode">
            <el-radio-group v-model="dataForm.editMode">
              <el-radio label="0" border>Markdown</el-radio>
              <el-radio label="1" border>HTML可视化</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="所属用户" prop="memberId">
            <el-input v-model="dataForm.memberName" placeholder="用户ID"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="置顶等级" prop="level">
            <el-input v-model="dataForm.level" placeholder="置顶等级"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="点赞人数" prop="voteUp">
            <el-input v-model="dataForm.voteUp" placeholder="点赞人数"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="点踩人数" prop="voteDown">
            <el-input v-model="dataForm.voteDown" placeholder="点踩人数"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="访问量" prop="viewCount">
            <el-input v-model="dataForm.viewCount" placeholder="访问量"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="评论数量" prop="commentCount">
            <el-input
              v-model="dataForm.commentCount"
              placeholder="评论数量"
            ></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="11">
          <el-form-item label="状态" prop="status">
            <el-select v-model="dataForm.status" clearable placeholder="请选择状态" style="width: 100%;">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="1">&nbsp;</el-col>
        <el-col :span="11">
          <el-form-item label="是否精华" prop="recommend">
            <el-radio-group v-model="dataForm.recommend">
              <el-radio :label="false" border>否</el-radio>
              <el-radio :label="true" border>是</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="内容" prop="content">
        <div v-html="dataForm.content" class="content"></div>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer" style="text-align: right;margin-top: 1em;">
      <el-button @click="visible = false">返回</el-button>
      <el-button v-if="dataForm.status === '0'" @click="examine(dataForm.id, 1)" type="success">审核通过</el-button>
      <el-button v-if="dataForm.status === '0'" @click="examine(dataForm.id, -1)" type="danger">审核不通过</el-button>
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
        title: '',
        coverImage: '',
        content: '',
        editMode: '0',
        categoryName: '',
        memberName: '',
        voteUp: '0',
        voteDown: '0',
        viewCount: '0',
        commentCount: '0',
        recommend: false,
        level: '0',
        status: '0'
      },
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
  methods: {
    examine(id, status) {
      this.$parent.examine(id, status, true)
    },
    closeArticleView() {
      this.visible = false
    },
    init(id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/article/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.title = data.article.title
              this.dataForm.coverImage = data.article.coverImage
              this.dataForm.content = data.article.content
              this.dataForm.editMode = data.article.editMode
              this.dataForm.categoryName = data.article.categoryName
              this.dataForm.memberName = data.article.memberName
              this.dataForm.voteUp = data.article.voteUp
              this.dataForm.voteDown = data.article.voteDown
              this.dataForm.viewCount = data.article.viewCount
              this.dataForm.commentCount = data.article.commentCount
              this.dataForm.recommend = data.article.recommend
              this.dataForm.level = data.article.level
              this.dataForm.status = data.article.status
            }
          })
        }
      })
    }
  }
}
</script>

<style>
.avatar {
  width: 100%;
  height: 300px;
  display: block;
}

/* table 样式 */
.content table {
  border-top: 1px solid #ccc;
  border-left: 1px solid #ccc;
}

.content table td,
.content table th {
  border-bottom: 1px solid #ccc;
  border-right: 1px solid #ccc;
  padding: 3px 5px;
}

.content table th {
  border-bottom: 2px solid #ccc;
  text-align: center;
}

/* blockquote 样式 */
.content blockquote {
  display: block;
  border-left: 8px solid #d0e5f2;
  padding: 5px 10px;
  margin: 10px 0;
  line-height: 1.4;
  font-size: 100%;
  background-color: #f1f1f1;
}

/* code 样式 */
.content code {
  display: inline-block;
  *display: inline;
  *zoom: 1;
  background-color: #f1f1f1;
  border-radius: 3px;
  padding: 3px 5px;
  margin: 0 3px;
}

.content pre code {
  display: block;
}

/* ul ol 样式 */
.content ul, .content ol {
  margin: 10px 0 10px 20px;
}
</style>
