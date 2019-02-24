<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" show-icon :closable="false"></el-alert>
      <el-row>
        <el-col>
          <span>选择商品分类:</span>
          <el-cascader
            expand-trigger="hover"
            :options="catInfos"
            :props="catInfosProps"
            v-model="selectedCat"
            @change="handleChange"
            change-on-select
            clearable
          ></el-cascader>
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="showButton">添加参数</el-button>
          <el-table :data="manyParamInfos" border style="width: 100%">
            <el-table-column type="expand">参数对应的值得部分</el-table-column>
            <el-table-column type="index" label="序号" width="60"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作" width="340">
              <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
              <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="showButton">添加属性</el-button>
          <el-table :data="onlyParamInfos" border style="width: 100%">
            <el-table-column type="expand">参数对应的值得部分</el-table-column>
            <el-table-column type="index" label="序号" width="60"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作" width="340">
              <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
              <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getCatInfos()
  },
  methods: {
    // 点击切换标签后的回调处理
    handleClick() {
      if (this.catThreeId === 0) {
        return null
      }
      this.getParamInfos()
    },

    async getParamInfos() {
      const { data: res } = await this.$http.get(
        'categories/' + this.catThreeId + '/attributes',
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      if (this.activeName === 'many') {
        this.manyParamInfos = res.data
      } else {
        this.onlyParamInfos = res.data
      }
    },

    // 商品分类级联选择器内容变化的回调
    handleChange() {
      if (this.selectedCat.length !== 3) {
        // 设置添加按钮为禁用状态
        this.showButton = true
        // 设置级联选择器内容为空
        this.selectedCat = []
        // 清空catThreeId成员
        this.catThreeId = 0

        this.manyParamInfos = []
        this.onlyParamInfos = []
      } else {
        // 设置添加按钮为解禁状态
        this.showButton = false
        // 把当前选取的第三级别分类的id赋予给catThreeId
        this.catThreeId = this.selectedCat[2]
        this.getParamInfos()
      }
    },
    // 获得3级商品信息列表
    async getCatInfos() {
      const { data: res } = await this.$http.get('categories', {
        param: { type: 3 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.catInfos = res.data
    }
  },

  data() {
    return {
      activeName: 'many', // 设置或接受选中标签项目的值

      catInfos: [], // 第1,2级别分类信息（供选取使用）
      // 下拉列表中给各个节点定义名称，值，子级
      catInfosProps: {
        // element-ui中定义的参数label，children，value
        label: 'cat_name',
        children: 'children',
        value: 'cat_id'
      },
      selectedCat: [], // 设定分类后的分类变量, 设定 或 接收 当前选中的项目

      showButton: true, // 设置添加参数按钮禁用

      catThreeId: 0,

      manyParamInfos: [],
      onlyParamInfos: []
    }
  }
}
</script>

<style lang="less" scoped>
</style>
