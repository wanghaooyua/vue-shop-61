<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <!-- 用户列表的主页内容 -->
      <el-table :data="rightsInfors" border style="width: 100%">
        <el-table-column type="index" label="序号" width="200"></el-table-column>
        <el-table-column prop="authName" label="名称" width="220"></el-table-column>
        <el-table-column prop="path" label="路径" width="210"></el-table-column>
        <el-table-column prop="level" label="等级">
          <template slot-scope="info">
            <el-tag v-if="info.row.level==0">标签一</el-tag>
            <el-tag v-else-if="info.row.level==1" type="success">标签二</el-tag>
            <el-tag v-else type="warning">标签四</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getRightsInfo()
  },
  methods: {
    async getRightsInfo() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightsInfors = res.data
    }
  },
  data() {
    return {
      rightsInfors: []
    }
  }
}
</script>

<style lang="less" scoped>
</style>
