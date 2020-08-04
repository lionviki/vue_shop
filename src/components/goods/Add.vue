<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <!-- Alert警告 -->
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
      </el-alert>
      <!-- Steps步骤条 -->
      <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form :model="addForm" ref="addFormRef" :rules="addFormRules" label-width="100px" label-position="top">
        <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTableave" @tab-click="tabClicked">
          <el-tab-pane label="基本信息" name="0">
            <!-- 基本信息部分表单 -->
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_number">
              <!-- 级联选择器 -->
              <el-cascader
                v-model="addForm.goods_cat"
                :options="catelist"
                :props="cascaderProps"
                @change="handleChange"
                clearable>
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染参数 -->
            <el-form-item :label="item.attr_name" v-for="item in manyTable" :key="item.attr_id">
              <!-- 渲染可选项 Checkbox多选框 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <!-- 渲染属性 -->
            <el-form-item :label="item.attr_name" v-for="item in onlyTable" :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- 图片上传upload区域 -->
            <!-- action 图片上传到后台的API地址 -->
            <el-upload
              action="http://127.0.0.1:8888/api/private/v1/upload"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headersObj"
              :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器区域 -->
            <quill-editor v-model="addForm.goods_introduce">
            </quill-editor>
            <!-- 添加商品确定按钮 -->
            <el-button class="btnAdd" type="primary" @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewDialogVisible"
      width="50%">
      <img :src="previewPath" alt="">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data() {
    return {
      // 步骤条当前激活部分
      activeIndex: '0',
      // 添加商品的表单数据
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品分类所属的数组
        goods_cat: [],
        // 图片数组
        pics: [],
        // 商品的详细描述
        goods_introduce: '',
        attrs: []
      },
      // 添加商品的表单验证
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      // 商品分类列表数据
      catelist: [],
      // 级联选择器的配置对象
      cascaderProps: {
        checkStrictly: false,
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 分类参数数据
      manyTable: [],
      // 商品属性数据
      onlyTable: [],
      // 图片上传请求头
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 图片预览路径
      previewPath: '',
      // 控制图片预览对话框显示隐藏
      previewDialogVisible: false
    }
  },
  created() {
    // 获取商品分类列表
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败!')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 级联选择器选项发生变化时，触发
    handleChange () {
      console.log(this.addForm.goods_cat)
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // 监听标签页的切换行为 activeName即将进去的标签页
    // oldActiveName即将离开的标签页
    beforeTableave (activeName, oldActiveName) {
      // 如果当前处在第一个标签页且级联选择器数组长度不为3没选中则阻止
      if (this.activeIndex === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    // tab栏的点击事件
    async tabClicked () {
      // 如果激活下标为1证明 当前处于商品参数页面
      if (this.activeIndex === '1') {
        // 发起获取商品参数请求
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (res.meta.status !== 200) {
          this.$message.error('获取分类参数失败!')
        }
        // 把每一项的attr_vals字符串转换为数组才能渲染
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTable = res.data
        console.log(this.manyTable)
      }
      if (this.activeIndex === '2') {
        // 发起获取商品属性请求
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) {
          this.$message.error('获取商品属性失败!')
        }
        this.onlyTable = res.data
        console.log(this.onlyTable)
      }
    },
    // 处理上传图片预览效果
    handlePreview (file) {
      // file下的response是服务器响应数据 file.response.data.url是图片真实路径
      // 存到data中 供对话框使用
      this.previewPath = file.response.data.url
      this.previewDialogVisible = true
    },
    // 处理移除图片操作
    handleRemove (file) {
      // 1. 获取将要删除图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2. 从pics数组中找到这个图片的索引值
      const i = this.addForm.pics.findIndex(x => x.pic === filePath)
      // 3. 调用数组的splice方法 把图片信息对象从pics数组中删除
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm)
    },
    // 监听图片上传成功后的操作
    handleSuccess (response) {
      // response服务器响应的数据
      // 拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 将图片信息对象push到pics数组中
      this.addForm.pics.push(picInfo)
      console.log(this.addForm)
    },
    // 添加商品功能
    add () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项!')
        }
        // 执行添加商品的业务逻辑
        // 先深拷贝一下添加商品所需要提交的表单对象addForm
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数 遍历manyTable每一项，创建新对象接收需要的两个属性
        // 在push到addForm添加表单中
        this.manyTable.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTable.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        // 把addForm的attrs属性复制到深拷贝的form中
        form.attrs = this.addForm.attrs
        console.log(form)

        // 发起请求 添加商品
        // 商品的名称必须是唯一的
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败!')
        }
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    // 获取级联选择器中的三级分类id
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      } else {
        return null
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}

.el-dialog img {
  width: 100%;
}

.btnAdd {
  margin-top: 15px;
}
</style>
