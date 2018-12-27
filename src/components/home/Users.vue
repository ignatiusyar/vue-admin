<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item to="/users">用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索功能 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <!-- 添加功能 -->
    <el-button type="success" plain>添加用户</el-button>
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
          <el-button type="primary" icon="el-icon-edit" circle></el-button>
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
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      userList: [],
      query: '',
      currentPage: 1,
      pageSize: 2,
      total: 0,
      whoami: '',
      baseURL: 'http://localhost:8888/api/private/v1/'
    }
  },
  created() {
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
    getUserList() {
      axios({
        method: 'get',
        url: this.baseURL + 'users',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        },
        headers: {
          Authorization: localStorage.getItem('token')
        }
      }).then(res => {
        // console.log(res.data)
        if (res.data.meta.status === 200) {
          this.userList = res.data.data.users
          this.total = res.data.data.total
        }
      })
    },
    changeState(user) {
      axios({
        method: 'put',
        url: this.baseURL + `users/${user.id}/state/${user.mg_state}`,
        headers: { Authorization: localStorage.getItem('token') }
      }).then(res => {
        if (res.data.meta.status === 200) {
          this.$message.success(
            res.data.data.mg_state === 1 ? '该用户已被启用!' : '该用户已被禁用!'
          )
          this.getUserList()
        } else {
          this.$message.error('网络错误,请稍后再试!')
          this.getUserList()
        }
      })
    },
    searchUser() {
      // 如果是搜索，应该让当前从第1页开始
      this.currentPage = 1
      this.getUserList()
    },
    delUser(id, username) {
      if (localStorage.getItem('whoami') === username) {
        this.$message.error('请勿删除当前用户')
        return false
      }
      this.$confirm('你确定要删除该用户吗?', '警告', {
        type: 'warning'
      })
        .then(() => {
          axios({
            method: 'delete',
            url: this.baseURL + `users/${id}`,
            headers: {
              Authorization: localStorage.getItem('token')
            }
          }).then(res => {
            if (res.data.meta.status === 200) {
              if (this.userList.length <= 1) {
                this.getUserList()
                this.$message.success('删除成功')
              } else {
                this.getUserList()
                this.$message.error('删除失败')
              }
            }
          })
        })
        .catch(() => {
          this.$message.info('取消删除')
        })
    }
  }
}
</script>

<style lang="less">
.el-breadcrumb {
  height: 40px;
  line-height: 40px;
}
.el-pagination {
  margin-top: 20px;
  text-align: right;
}
.el-input {
  width: 300px;
  margin-bottom: 5px;
}
.el-table {
  margin-top: 10px;
}
</style>
