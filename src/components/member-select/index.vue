<template>
  <el-select
    style="width: 100%"
    v-model="selectValue"
    :multiple="multiple"
    :reserve-keyword="multiple"
    filterable
    remote
    placeholder="请输入ID/用户名"
    :remote-method="remoteMethod"
    :loading="loading"
  >
    <el-option
      v-for="item in options"
      :key="item.id"
      :label="item.memberName"
      :value="item.id"
    >
    </el-option>
  </el-select>
</template>

<script>
import { notEmpty } from '@/utils'

export default {
  name: 'member-select',
  props: {
    multiple: {
      type: Boolean,
      default: false
    },
    value: {
      type: [Array, String],
      default: []
    }
  },
  data() {
    return {
      options: [],
      selectValue: this.value,
      loading: false
    }
  },
  // TODO 因为前端传数组参数会带上'[]'，所以暂时使用POST请求方式，可以在前端使用qs序列化数组后传入
  methods: {
    closeSelect() {
      this.options = []
      this.selectValue = this.multiple ? [] : null
    },
    init(ids) {
      if (ids) {
        this.$http({
          url: this.$http.adornUrl('/admin/member/select/search'),
          method: 'post',
          data: this.$http.adornData({ids: Array.isArray(ids) ? ids : [ids]}, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.options = data.list
            this.selectValue = this.multiple ? data.list.map(item => item.id) : data.list[0].id
          }
        })
      }
    },
    remoteMethod(query) {
      if (query) {
        this.loading = true
        // 发送请求查询指定用户
        this.$http({
          url: this.$http.adornUrl('/admin/member/select/search'),
          method: 'post',
          data: this.$http.adornData({query}, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.options = data.list
            this.loading = false
          } else {
            this.$message.error(data.msg)
            this.options = []
          }
        })
      } else {
        this.options = []
      }
    }
  },
  watch: {
    value(newVal) {
      if (notEmpty(newVal, true) && !this.multiple) {
        this.selectValue = newVal[0]
      } else if ((notEmpty(newVal, true) && this.multiple) || (!Array.isArray(newVal) && !this.multiple)) {
        this.selectValue = newVal
      } else if (!Array.isArray(newVal) && this.multiple) {
        this.selectValue = [newVal]
      }
    },
    selectValue(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.$emit('input', this.selectValue)
      }
    }
  }
}
</script>
