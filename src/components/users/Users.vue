<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item to="/users">用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索功能 -->
    <el-input
      placeholder="请输入内容"
      v-model="query"
      class="input-with-select"
      @keyup.enter.native="searchUser"
    >
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <!-- 添加功能 -->
    <el-button type="success" plain @click="showAdd">添加用户</el-button>
    <!-- 表格 -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column prop="username" label="姓名" width="250"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="250"></el-table-column>
      <el-table-column prop="mobile" label="电话" width="250"></el-table-column>
      <el-table-column prop="role_name" label="角色" width="200"></el-table-column>
      <el-table-column prop="mg_state" label="状态" width="250">
        <template slot-scope="scope">
          <!-- {{scope.row}} -->
          <el-switch
            @change="changeState(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :disabled="scope.row.username === whoami"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop label="操作">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" circle @click="showEdit(scope.row)"></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            @click="delUser(scope.row.id,scope.row.username)"
            v-if="scope.row.username !== whoami && scope.row.id !== 500"
          ></el-button>
          <el-button type="warning" round icon="el-icon-view" circle></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!--
      分页插件
       size-change: 每页的条数发生了改变，会触发
       current-change: 当前页发生了改变，会触发
       current-page： 当前页
       page-size: 每页的条数
       total: 总条数
       page-sizes="[2, 4, 6, 8]： 设置修改每页条数的选项
       layout: 控制分页的布局
    -->
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 4, 6, 8, 10]"
      :page-size="pageSize"
      :total="total"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- 添加对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="30%">
      <el-form ref="addForm" :model="addForm" label-width="80px" status-icon :rules="rules">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入邮箱地址"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入电话号码"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="30%">
      <el-form ref="editForm" :model="editForm" label-width="80px" status-icon :rules="rules">
        <el-form-item label="用户名">
          <el-tag type="info">{{editForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" placeholder="请输入邮箱地址"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="editForm.mobile" placeholder="请输入电话号码"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="EditUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userList: [],
      query: '',
      currentPage: 1,
      pageSize: 2,
      total: 0,
      whoami: '',
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
        username: '',
        password: '',
        email: '',
        mobile: '',
        id: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不得为空', trigger: 'change' },
          {
            min: 2,
            max: 9,
            message: '用户名长度应在2~9位之间',
            trigger: 'change'
          }
        ],
        password: [
          { required: true, message: '密码不得为空', trigger: 'change' },
          {
            min: 6,
            max: 12,
            message: '密码应在6-12位之间',
            trigger: 'change'
          }
        ],
        email: [
          { type: 'email', message: '请输入一个合法的邮箱', trigger: 'change' }
        ],
        mobile: [
          {
            pattern: /^(0|86|17951)?(13[0-9]|15[012356789]|166|17[3678]|18[0-9]|14[57])[0-9]{8}$/,
            message: '请输入一个合法的手机号',
            trigger: 'change'
          }
        ]
      }
    }
  },
  beforeMount() {
    this.whoami = localStorage.getItem('whoami')
    this.getUserList()
  },
  methods: {
    handleSizeChange(val) {
      this.pageSize = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.getUserList()
    },
    // 获取数据
    async getUserList() {
      let res = await this.axios({
        method: 'get',
        url: 'users',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      })
      let {
        meta: { status },
        data: { users, total }
      } = res.data
      // console.log(res.data)
      if (status === 200) {
        this.userList = users
        this.total = total
      }
    },
    // 改变用户状态
    async changeState({ id, mg_state: mgState }) {
      // { id,mg_state }
      let res = await this.axios({
        method: 'put',
        url: `users/${id}/state/${mgState}`
      })
      if (res.data.meta.status === 200) {
        this.$message.success(
          res.data.data.mg_state === 1 ? '该用户已被启用!' : '该用户已被禁用!'
        )
        this.getUserList()
      } else {
        this.$message.error('网络错误,请稍后再试!')
      }
    },
    // 搜索功能
    searchUser() {
      // 如果是搜索，应该让当前从第1页开始
      this.currentPage = 1
      this.getUserList()
    },
    // 删除用户
    async delUser(id, username) {
      if (localStorage.getItem('whoami') === username) {
        this.$message.error('请勿删除当前用户')
        return false
      }
      try {
        await this.$confirm('你确定要删除该用户吗?', '警告', {
          type: 'warning'
        })
        let res = await this.axios({
          method: 'delete',
          url: `users/${id}`
        })
        if (res.data.meta.status === 200) {
          if (this.userList.length <= 1) {
            this.currentPage--
          }
          this.getUserList()
          this.$message.success('删除成功')
        } else {
          this.getUserList()
          this.$message.error(res.data.meta.msg)
        }
      } catch (e) {
        this.$message.info('取消删除')
      }
    },
    // 显示添加模态框
    showAdd() {
      this.addDialogVisible = true
    },
    // 添加用户
    async addUser() {
      try {
        await this.$refs.addForm.validate()
        let res = await this.axios({
          method: 'post',
          url: 'users',
          data: this.addForm
        })
        if (res.data.meta.status === 201) {
          this.$refs.addForm.resetFields()
          this.addDialogVisible = false
          // 重新渲染页面
          this.total++
          this.currentPage = Math.ceil(this.total / this.pageSize)
          this.getUserList()
          this.$message.success('用户添加成功')
        } else {
          this.$message.error(res.data.meta.msg)
        }
      } catch (e) {
        return false
      }
    },
    // 显示编辑用户模态框
    showEdit(user) {
      this.editDialogVisible = true
      // this.editForm = user
      this.editForm.username = user.username
      this.editForm.email = user.email
      this.editForm.mobile = user.mobile
      this.editForm.id = user.id
    },
    // 编辑用户
    async EditUser() {
      try {
        await this.$refs.editForm.validate()
        let res = await this.axios({
          method: 'put',
          url: `users/${this.editForm.id}`,
          data: this.editForm
        })
        if (res.data.meta.status === 200) {
          this.$refs.editForm.resetFields()
          this.editDialogVisible = false
          this.getUserList()
          this.$message.success('用户信息修改成功')
        } else {
          this.$message.error(res.data.meta.msg)
        }
      } catch (e) {
        return false
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-breadcrumb {
  height: 40px;
  line-height: 40px;
}
.el-pagination {
  margin-top: 20px;
  text-align: right;
}
.input-with-select {
  width: 300px;
  margin-bottom: 5px;
}
.el-table {
  margin-top: 10px;
}
</style>
