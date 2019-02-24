<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-row>
        <el-button type="primary">添加角色</el-button>
      </el-row>
      <el-table :data="roleInfos" border style="width: 100%">
        <!-- type: 对应列的类型。如果设置了 selection 则显示多选框；如果设置了 index 则显示该行的索引（从 1 开始计算）；
        如果设置了 expand 则显示为一个可展开的按钮-->
        <el-table-column type="expand" width="60">
          <template slot-scope="info">
            <el-row
              v-for="(item,k) in info.row.children"
              :key="item.id"
              @close="closeTag(info.row.id,item.id)"
              :style="{'border-bottom':'1px solid #EBEEF5','border-top':k===0?'1px solid #EBEEF5':''}"
            >
              <el-col :span="5">
                <el-tag closable>{{item.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  v-for="(item2,kk) in item.children"
                  :key="item2.id"
                  @close="closeTag(info.row.id,item2.id)"
                  :style="{'border-top':kk!==0?'1px solid #EBEEF5':''}"
                >
                  <el-col :span="8">
                    <el-tag type="success" closable>{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="16">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      @close="closeTag(info.row.id,item3.id)"
                      :key="item3.id"
                      closable
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="序号" width="80"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述" width="180"></el-table-column>
        <el-table-column label="操作" width="330">
          <template slot-scope="info">
            <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
            <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
            <el-button
              type="warning"
              size="mini"
              icon="el-icon-setting"
              @click="showDistributeDialog(info.row)"
            >操作权限</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分配权限的弹框 rules:element-ui中内置的表单验证规则-->
      <el-dialog title="分配权限" :visible.sync="distributeDialogVisible" width="50%">
        <el-form ref="distributeFormRef" :model="distributeForm" label-width="120px">
          <el-form-item label="角色分配">{{distributeForm.roleName}}</el-form-item>
          <el-form-item>
            <el-tree
              :data="rightsInfo"
              :props="rightsInfoProps"
              node-key="id"
              show-checkbox
              default-expand-all
              :default-checked-keys="deFaultCheckedKeys"
              ref="rightsTree"
            ></el-tree>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="distributeDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="distributeRights">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    // 生命周期函数调用页面数据函数显示
    this.getRoleInfos()
  },
  methods: {
    // 确定分配权限按钮
    async distributeRights() {
      // 获取全选中的节点的id信息
      var ids1 = this.$refs.rightsTree.getCheckedKeys()
      // 获得半选中的节点的id信息
      var ids2 = this.$refs.rightsTree.getHalfCheckedKeys()
      // 把全选和半选的id信息拼接成用逗号分割的字符串
      var allIds = [...ids1, ...ids2].join(',')

      if (!allIds) {
        return this.$message.error('请选择权限')
      }

      const { data: res } = await this.$http.post(
        'roles/' + this.distributeForm.id + '/rights',
        { rids: allIds }
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.distributeDialogVisible = false
      this.$message.success(res.meta.msg)
      this.getRoleInfos()
    },

    // 分配权限按钮
    async showDistributeDialog(role) {
      this.distributeDialogVisible = true
      // 把role数据赋予给distributeForm表单数据对象
      this.distributeForm = role

      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightsInfo = res.data
      var idArr = []
      this.getHaveRights(role, idArr)
      this.deFaultCheckedKeys = idArr
    },
    getHaveRights(node, keys) {
      if (!node.children) {
        return keys.push(node.id)
      }
      node.children.forEach(item => {
        return this.getHaveRights(item, keys)
      })
    },

    // 点击叉号删除角色指定权限操作
    closeTag(roleId, rightsId) {
      this.$confirm('确认删除该权限?', '删除权限', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async() => {
          const { data: res } = await this.$http.delete(
            'roles/' + roleId + '/rights/' + rightsId
          )
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success(res.meta.msg)
          this.getRoleInfos()
        })
        .catch(() => {})
    },

    // 获取页面数据
    async getRoleInfos() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.roleInfos = res.data
      console.log(this.roleInfos)
    }
  },
  data() {
    return {
      distributeDialogVisible: false,
      distributeForm: {
        id: 0,
        roleName: '' // 当前被分配角色的角色名称
      },
      // rightsInfo：接收用于分配的树形权限信息
      rightsInfo: [],
      rightsInfoProps: {
        // 给大树设置数据属性
        label: 'authName', // 设定"树"中数据
        children: 'children' // 给大树 设置数据属性，例如显示的名称、起作用的值的部分
      },
      deFaultCheckedKeys: [], // 设定树节点默认选中情况

      // 数据展示列表
      roleInfos: []
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 10px 5px;
}
.el-row {
  display: flex;
  align-items: center;
}
</style>
