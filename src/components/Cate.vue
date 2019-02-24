<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-button type="primary" @click="showAddDialog">添加分类</el-button>

      <!-- 添加用户的弹框 rules:element-ui中内置的表单验证规则-->
      <el-dialog title="添加分类" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
        <el-form :rules="addFromRules" ref="addFormRef" :model="addForm" label-width="80px">
          <el-form-item label="活动名称" prop="cat_name">
            <el-input v-model="addForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="分类上级">
            <el-cascader
              expand-trigger="hover"
              :options="catInfosTwo"
              :props="catInfosProps"
              v-model="selectedCat"
              @change="handleChange"
              change-on-select
              clearable
            ></el-cascader>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCat">确 定</el-button>
        </span>
      </el-dialog>

      <zk-table
        :data="cateInfos"
        :columns="cateInfoscolumns"
        :selection-type="false"
        :expand-type="false"
        show-index
      >
        <template slot="order" slot-scope="info">
          <el-tag v-if="info.row.cat_level==0">一级</el-tag>
          <el-tag v-else-if="info.row.cat_level==1">二级</el-tag>
          <el-tag v-else>三级</el-tag>
        </template>
        <template slot="opt">
          <el-button type="primary" size="mini" class="el-icon-edit">编辑</el-button>
          <el-button type="danger" size="mini" class="el-icon-delete">删除</el-button>
        </template>
      </zk-table>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getCteInfos()
  },
  methods: {
    // 点击确定添加分类按钮
    addCat() {
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post(
            'categories',
            this.addForm
          )
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }
          this.addDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getCteInfos()
        }
      })
    },
    // 点击添加分类按钮
    async showAddDialog() {
      this.addDialogVisible = true
      // 获得级联选择器第1,2级需要的数据
      const { data: res } = await this.$http.get('categories', {
        // 传入type：2 意思为获得到第二级的数据
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.catInfosTwo = res.data
    },

    // 关闭添加分类弹框
    addDialogClose() {
      // 重置form表单
      this.$refs.addFormRef.resetFields()
      // 上级分类 级联选择器 也需要重置
      this.selectedCat = []
    },
    // 级联选择器北荣变化时出发的回调函数
    handleChange() {
      console.log(this.selectedCat)
      if (this.selectedCat.kenght === 0) {
        this.addForm.cat_pid = 0
        this.addForm.cat_level = 0
      } else {
        var len = this.selectedCat.length
        this.addForm.cat_pid = this.selectedCat[len - 1]
        this.addForm.cat_level = len
      }
    },

    // 显示列表
    async getCteInfos() {
      const { data: res } = await this.$http.get('categories', {
        params: {
          type: 3,
          pagenum: this.queryParams.pageNum,
          pagesize: this.queryParams.pageSize
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateInfos = res.data.result
      console.log(res)
    }
  },
  data() {
    return {
      // 添加分类弹框显示
      addDialogVisible: false,
      // form表单验证规则
      addFromRules: {
        cat_name: [{ required: true, message: '分类名称必填', trigger: 'blur' }]
      },
      // 弹出框表单数据显示内容
      addForm: {
        cat_name: '', // 分类名称
        cat_pid: 0, // 分类上级id
        cat_level: 0 // 分类级别
      },

      // 级联选择器相关成员(下拉列表)
      catInfosTwo: [], // 第1,2级别分类信息（供选取使用）
      selectedCat: [], // 设定分类后的分类变量, 设定 或 接收 当前选中的项目的cat_pid
      // 下拉列表中给各个节点定义名称，值，子级
      catInfosProps: {
        // element-ui中定义的参数label，children，value
        label: 'cat_name',
        children: 'children',
        value: 'cat_id'
      },

      queryParams: {
        pageNum: 1, // 显示第几页
        pageSize: 5, // 每页显示条数
        total: 0 // 记录总条数
      },
      cateInfos: [],
      cateInfoscolumns: [
        { label: '分类名称', prop: 'cat_name' },
        { label: '排序', type: 'template', template: 'order' },
        { label: '操作', type: 'template', template: 'opt' }
      ]
    }
  }
}
</script>

<style lang="less" scoped>
.zk-table {
  margin-top: 15px;
}
</style>
