<template>
  <el-container class="home">
    <el-header>
      <div class="logo"></div>
      <div class="title">
        <h1></h1>
      </div>
      <div class="logstatus">
        <span>Dear {{ username }}，Welcome Back</span>
        <i>|</i>
        <a href="javascript:;" @click="logout">Logout</a>
      </div>
    </el-header>
    <el-container>
      <el-aside width="200px">
        <el-menu
          default-active="/home"
          class="el-menu-vertical-demo"
          background-color="#254673"
          text-color="#fff"
          active-text-color="#ffd04b"
          unique-opened
          router
        >
          <el-submenu index="1">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>用户管理</span>
            </template>
            <el-menu-item index="/users">
              <i class="el-icon-menu"></i>
              <span slot="title">用户列表</span>
            </el-menu-item>
          </el-submenu>
          <el-submenu index="2">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>权限管理</span>
            </template>
            <el-menu-item index="/roles">
              <i class="el-icon-menu"></i>
              <span slot="title">角色列表</span>
            </el-menu-item>
            <el-menu-item index="/rights">
              <i class="el-icon-menu"></i>
              <span slot="title">权限列表</span>
            </el-menu-item>
          </el-submenu>
          <el-submenu index="3">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>导航一</span>
            </template>
            <el-menu-item index="1-1">选项1</el-menu-item>
            <el-menu-item index="1-2">选项2</el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view/>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      username: ''
    }
  },
  created() {
    this.username = localStorage.getItem('whoami')
  },
  methods: {
    // 退出功能
    async logout() {
      await this.$confirm('您确定要退出本系统吗?', '系统提示', {
        type: 'warning'
      })
      try {
        localStorage.removeItem('token')
        localStorage.removeItem('whoami')
        this.$router.push('/login')
        this.$message.success('您已成功退出系统')
      } catch (e) {
        this.$message.info('取消操作成功')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.home {
  height: 100%;
  .el-header {
    // background: #409eff;
    background-color: #192f51;
    // border-bottom: 1px solid #c0c0c0;
    display: flex;
    .logo {
      width: 180px;
      margin-left: -10px;
      background-image: url('~@/assets/logo2.png');
      background-size: contain;
      background-repeat: no-repeat;
      background-position: center center;
    }
    .title {
      text-align: center;
      line-height: 60px;
      flex: 1;
      color: #fff;
      font-size: 18px;
    }
    .logstatus {
      line-height: 60px;
      color: #fff;
      a {
        color: #fff;
        opacity: 0.7;
        text-decoration: none;
      }
      i {
        font-size: 13px;
        font-style: normal;
        color: #ccc;
        text-align: middle;
      }
    }
  }
  .el-aside {
    // background-color: #545c64;
    background-color: #254673;
    width: 200px;
    .el-submenu {
      width: 200px;
    }
  }
  .el-main {
    background-color: #f0f0f0;
    // background-color: #fff;
  }
}
</style>
