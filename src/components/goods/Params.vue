<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告框 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数!" type="warning" :closable="false" show-icon>
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat-opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择器 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cascaderProps"
            @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>
      <!-- tabs 页签区域 -->
      <el-tabs v-model="activeName" type="card" @tab-click="handleTabClick">
        <!-- 动态参数 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
        </el-tab-pane>
        <!-- 静态属性 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled">添加属性</el-button>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有商品分类列表
      catelist: [],
      // 级联选择框双向绑定的数据 必须为数组
      selectedCateKeys: [],
      // 级联选择器设置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // tabs标签页当前激活
      activeName: 'many',
      // 动态表格的数据
      manyTableData: [],
      // 静态表格的数据
      onlyTableData: []
    }
  },
  created() {
    // 获取所有商品分类列表
    this.getCateList()
  },
  methods: {
    // 获取所有商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败!')
      }
      this.catelist = res.data
    },
    // 级联选择器选中项发生变化 会触发这个函数
    handleChange () {
      // 调用获取参数列表函数
      this.getParamsData()
    },
    // tabs标签页 点击的处理函数
    handleTabClick () {
      console.log(this.activeName)
      // tab切换时也要发送获取参数列表请求的函数
      this.getParamsData()
    },
    // 抽离出来的获取参数列表的函数
    async getParamsData () {
      // 如果级联选择器双向绑定的数组的长度不为3 则不让其选中
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        return
      }
      // 发起获取参数列表的请求
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败!')
      }
      // console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
        console.log(this.manyTableData)
      } else {
        this.onlyTableData = res.data
        console.log(this.onlyTableData)
      }
    }
  },
  computed: {
    // 是否让button按钮禁用
    isBtnDisabled () {
      // 如果级联选择器未选中 则让btn处于禁用状态返回true
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 获取级联选择器选中的第三级分类的Id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.cat-opt {
  margin: 15px 0;
}
</style>
