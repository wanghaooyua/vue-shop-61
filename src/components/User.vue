<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <!-- Row 组件 提供 gutter 属性来指定每一栏之间的间隔，默认间隔为。 -->
      <el-row :gutter="20">
        <!-- 通过 col 组件的 span 属性我们就可以自由地组合布局。 -->
        <el-col :span="8">
          <el-input
            placeholder="请输入要搜索的内容"
            v-model="queryParams.query"
            :clearable="true"
            @clear="getUserInfors"
            @keyup.enter.native="getUserInfors"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfors"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 用户列表的主页内容 -->
      <el-table :data="userInfors" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名" width="250"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="140"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="180"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="150"></el-table-column>
        <el-table-column label="状态" width="80">
          <el-switch
            v-model="info.row.mg_state"
            slot-scope="info"
            @change="changeState(info.row.id),info.row.mg_state"
          ></el-switch>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="info">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(info.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="delUser(info.row.id)"
            ></el-button>
            <el-tooltip content="分配角色" placement="top" :enterable="false">
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
                @click="showFenpeiDialog(info.row.id)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!--数据分页展示区域-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryParams.pagenum"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="queryParams.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="queryParams.total"
      ></el-pagination>

      <!-- 添加用户的弹框 rules:element-ui中内置的表单验证规则-->
      <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
        <el-form :rules="addFromRules" ref="addFormRef" :model="addForm" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户的弹框 rules:element-ui中内置的表单验证规则-->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <el-form :rules="editFromRules" ref="editFormRef" :model="editForm" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分配角色的弹框-->
      <el-dialog
        title="用户分配角色"
        :visible.sync="fenpeiDialogVisible"
        width="50%"
        @close="fenpeiDialogClose"
      >
        <el-form
          :rules="fenpeiFormRoles"
          ref="fenpeiFormRef"
          :model="fenpeiForm"
          label-width="130px"
        >
          <el-form-item label="当前的用户" prop="username">{{fenpeiForm.username}}</el-form-item>
          <el-form-item label="分配新角色" prop="rid">
            <el-select v-model="fenpeiForm.rid" placeholder="请选择">
              <el-option
                v-for="item in roleInfos"
                :key="item.id"
                :label="item.roleName"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="fenpeiDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="fenpeiUser">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getUserInfors()
  },
  methods: {
    // 分配用户权限
    // 关闭弹框后重置form表单
    fenpeiDialogClose() {
      this.$refs.fenpeiFormRef.resetFields()
    },
    async fenpeiUser(id) {
      const { data: res } = await this.$http.put(
        'users/' + this.fenpeiForm.id + '/role',
        this.fenpeiForm
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.fenpeiDialogVisible = false
      this.$message.success(res.meta.msg)
      this.getUserInfors()
    },
    // 点击修改权限按钮弹出弹框
    async showFenpeiDialog(id) {
      this.fenpeiDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.fenpeiForm = res.data
      const { data: res2 } = await this.$http.get('roles')
      if (res2.meta.status !== 200) {
        return this.$message.error(res2.meta.msg)
      }
      this.roleInfos = res2.data
    },

    // 修改用户信息
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.put(
            'users/' + this.editForm.id,
            this.editForm
          )
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.editDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfors()
        }
      })
    },

    // 删除用户
    delUser(id) {
      this.$confirm('确认删除该会员?', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async() => {
          // axios 触发api接口删除
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          this.$message.success(res.meta.msg)
          // 当删除一个页面只有一个用户时候，比如，
          // 第二页只有一个用户，当把这个用户删除之后，调用重新加载用户列表的函数，页面直接显示空数据了
          // 此时，页面重新加载数据时，对应的当前页码条件参数的值仍然为"2",而第2页已经没有数据了，所以页面显示空白
          // 解决：
          // 判断当前页码如果只有一条数据并且不是第1页，并且这个数据还被删除了，那么重新加载数据时，页码请给予减少1位操作
          if (this.userInfors.length === 1 && this.queryParams.pagenum > 1) {
            // this.userInfors.length  打印的是当前页面删除前有几个数据
            this.queryParams.pagenum--
          }
          this.getUserInfors()
        })
        .catch(() => {})
    },

    // 添加用户
    addUser() {
      // validate:对整个表单进行校验的方法，参数为一个回调函数。该回调函数会在校验结束后被调用，并传入两个参数：
      // 是否校验成功和未通过校验的字段。若不传入回调函数，则会返回一个 promise
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }

          // 关闭dialog弹框 提示成功的消息 重新加载数据列表
          this.addDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfors()
        }
      })
    },

    // 重置dialog表单
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },

    handleSizeChange(val) {
      // val:代表变化后的每页信息显示条数
      this.queryParams.pagesize = val
      // 根据变化后的 显示条数 重新获得数据
      this.getUserInfors()
    },
    handleCurrentChange(val) {
      // val:代表变化后的 页码
      this.queryParams.pagenum = val
      // 根据变化后的 页码 重新获得数据
      this.getUserInfors()
    },
    async changeState(id, state) {
      const { data: res } = await this.$http.put(`users/${id}/state/${state}`)
      if (res.meta.status !== 200) {
        return this.$message.error('修改状态失败')
      }
      this.$message({
        message: '修改状态成功',
        type: 'success',
        duration: 1500
      })
    },
    // 获得首屏用户数据
    async getUserInfors() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryParams
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 丰富记录总条数变量
      this.queryParams.total = res.data.total
      // 把获得的用户数据 赋予给userInfos成员
      this.userInfors = res.data.users
    }
  },
  data() {
    // 自定义校验器 rule不用，value被校验数据，callback回调函数
    var ckeckMobile = (rule, value, callback) => {
      if (!/^1[35789]\d{9}$/.test(value)) {
        return callback(new Error('手机号码格式不正确'))
      }
      callback()
    }
    return {
      // 分配用户角色
      // Dialog弹框是否显示，false：隐藏
      fenpeiDialogVisible: false,
      roleInfos: [],
      fenpeiForm: {
        username: '',
        rid: ''
      },
      fenpeiFormRoles: {
        rid: [{ required: true, message: '角色必须选择', trigger: 'change' }]
      },

      // Dialog弹框是否显示，false：隐藏
      editDialogVisible: false,
      // 修改用户的数据表单部分
      editForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // Dialog弹框表单验证规则
      editFromRules: {
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          // validator:自定义校验器,trigger:'blur'
          { validator: ckeckMobile, trigger: 'blur' }
        ]
      },

      // Dialog弹框是否显示，false：隐藏
      addDialogVisible: false,
      // 添加用户的数据表单部分
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // Dialog弹框表单验证规则
      addFromRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          // validator:自定义校验器,trigger:'blur'
          { validator: ckeckMobile, trigger: 'blur' }
        ]
      },

      // 接收用户数据
      userInfors: [],
      // 定义获取用户数据时用到的查询条件参数
      queryParams: {
        query: '',
        pagenum: 1, // 当前页码
        pagesize: 3, // 每页显示条数
        total: 0 // 总条数
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
