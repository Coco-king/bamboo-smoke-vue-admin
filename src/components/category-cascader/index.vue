<template>
  <el-cascader
    placeholder="请选择分类"
    style="width:100%"
    v-model="cascadeData"
    :options="categoryList"
    :props="props"
    @change="handleChange"
    clearable
    filterable
  ></el-cascader>
</template>

<script>
import { treeDataTranslate } from '@/utils'

export default {
  name: 'categoryCascader',
  props: {},
  data() {
    return {
      cascadeData: [],
      categoryList: [],
      props: {
        value: 'id',
        checkStrictly: true,
        label: 'name',
        children: 'children',
        expandTrigger: 'hover'
      }
    }
  },
  created() {
    this.init()
  },
  methods: {
    init() {
      this.resetFields()
      this.$http({
        url: this.$http.adornUrl('/api/category/list'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.categoryList = treeDataTranslate(data.list)
        }
      })
    },
    initSelect(categoryId) {
      this.$http({
        url: this.$http.adornUrl(
            '/api/category/path'
          ),
        method: 'get',
        params: this.$http.adornParams({
          id: categoryId
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.cascadeData = data.cascadeData
          this.handleChange(this.cascadeData)
        }
      })
    },
    handleChange(value) {
      this.$emit('input', value[value.length - 1])
      this.$emit('change', value)
    },
    resetFields() {
      this.cascadeData = []
    }
  }
}
</script>

