<template>
  <div class="users">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable @clear="_getUserList" v-model="queryInfo.query" class="input-with-select">
            <el-button slot="append" icon="el-icon-search" @click="_getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>

      <el-table
        :data="userlist"
        border
        style="width: 100%">
        <el-table-column
          type="index"
          label="索引"
          width="180">
        </el-table-column>
        <el-table-column
          prop="username"
          label="姓名"
          width="180">
        </el-table-column>
        <el-table-column
          prop="email"
          label="邮箱">
        </el-table-column>
        <el-table-column
          prop="mobile"
          label="电话">
        </el-table-column>
        <el-table-column
          prop="role_name"
          label="角色"
          width="180">
        </el-table-column>
        <el-table-column
          label="状态">
          <template slot-scope="scope">
            <el-switch @change="userStateChanged(scope.row)" v-model="scope.row.mg_state"></el-switch>
          </template>
        </el-table-column>
        <el-table-column
          label="操作" width="180px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showRole(scope.row)">
              分配角色
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 3, 4, 5]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
      >
      <!-- 内容主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="120px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
      >
      <!-- 内容主体区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="分配角色" :visible.sync="showRoleVisible" width="50%" @close="setRoleDialogClosed">
      <div>
        <p>当前的用户: {{userInfo.username}}</p>
        <p>当前的角色: {{userInfo.role_name}}</p>
      </div>

      <el-select v-model="roleId" placeholder="请选择">
        <el-option
          v-for="item in rolelist"
          :key="item.id"
          :label="item.roleName"
          :value="item.id">
        </el-option>
      </el-select>

      <span slot="footer" class="dialog-footer">
        <el-button @click="showRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRole">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/

        if (regEmail.test(value)) {
          return cb()
        }
        cb(new Error('请输入合法的邮箱'))
    }

    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
  
      if (regMobile.test(value)) {
        return cb()
      }

      cb(new Error('请输入合理的手机号'))
    }

    const checkPassword = (rule, value, cb) => {
      const regPassword = /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,20}$/
      
      if (regPassword.test(value)) {
        return cb()
      }
      return cb(new Error('密码包含数字和英文至少6个字符'))
    }

    return {
      list: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required:true, message:'请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名的长度在3-10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required:true, message:'请输入密码', trigger: 'blur' },
          {
            // min: 6,
            // max: 15,
            // message: '密码的长度在6-15个字符之间',
            validator: checkPassword,
            trigger: 'blur'
          }
        ],
        email: [
          { required:true, message:'请输入邮箱', trigger: 'blur' },
          {validator: checkEmail, trigger: 'blur'}
        ],
        mobile: [
          { required:true, message:'请输入手机号', trigger: 'blur' },
          {validator: checkMobile, trigger: 'blur'}
        ],
      },
      editFormRules: {
        email: [
          {required: true, message: '请输入手机号', trigger: 'blur'},
          {validator: checkEmail, trigger: blur}
        ]
      },
      editDialogVisible: false,
      editForm: {},
      showRoleVisible: false,
      rolelist: [],
      roleId: '',
      userInfo: {}
    }
  },
  created () {
    this._getUserList()
  },
  methods: {
    async getRolesList () {
      const {data: res} = await this.$http.get('/roles')

      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    },
    async _getUserList () {
      const {data: res} = await this.$http.get('/users', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      this.total = res.data.total
      this.userlist = res.data.users
      console.log(this.userlist)
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this._getUserList()
    },
    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage
      this._getUserList()
    },
    async userStateChanged(userinfo) {
      const {data: res} = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      console.log(res)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新状态失败')
      }
      this.$message.success('更新状态成功')
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      console.log('1')
      this.$refs.addFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const {data: res} = await this.$http.post('/users', 
          this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加失败！')
        }
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
        this._getUserList()
      })
    },
    async showEditDialog (id) {
      const {data: res} = await this.$http.get('/users/' + id)

      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
      console.log(this.editForm)
      this.editDialogVisible = true
    },
    editDialogClosed () {
      console.log('你')
      this.$refs.editFormRef.resetFields()
    },
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const {data:res} = await this.$http.put('/users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户失败！')
        }
        this.editDialogVisible = false
        this._getUserList()
        this.$message.success('更新用户成功！')
      })
    },
    async removeUserById (id) {
      console.log(this.userlist)
      const confirmResult = await this.$confirm(
        '此操作将永久删除用户，是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)

      if(confirmResult !== 'confirm') {
        return this.$message.info('已经取消了删除')
      }
      const {data: res} = await this.$http.delete('/users/' + id)
      
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }

      const {data: dres} = await this.$http.get('/users', {
        params: this.queryInfo
      })
      if (dres.meta.status !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      this.total = dres.data.total
      this.userlist = dres.data.users
      console.log(this.userlist)
      
      if(this.userlist.length === 0) {
        console.log('空')
        this.queryInfo.pagenum--
        console.log(this.queryInfo.pagenum)
        this._getUserList()
        console.log(this.userlist)
      } else {
        this._getUserList()
        console.log(this.userlist)
      }
      this.$message.success('删除用户成功！')
    },
    showRole (userInfo) {
      this.getRolesList()
      this.userInfo = userInfo
      console.log(userInfo)
      this.showRoleVisible = true
    },
    async setRole () {
      const {data: res} = await this.$http.put(`/users/${this.userInfo.id}/role`,{
        rid: this.roleId
      })
      console.log(res)
      console.log(this.roleId)
      if (res.meta.status !== 200) {
        return this.$message.error('设置角色失败！')
      }

      this.$message.success('设置角色成功！')
      this._getUserList()
      this.showRoleVisible = false
    },
    setRoleDialogClosed () {
      this.userInfo = {}
      this.roleId = ''
    }
  }
}
</script>

<style>

</style>