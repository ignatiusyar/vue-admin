<template>
  <div class="login">
    <!-- <el-button type="success">成功按钮</el-button> -->
    <div class="wrapper"></div>
    <el-form ref="form" :model="form" label-width="80px" :rules="rules" status-icon>
      <img src="@/assets/avatar.jpg" alt>
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="form.password" placeholder="请输入用户密码" type="password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">登录</el-button>
        <el-button @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'change' },
          { min: 3, max: 9, message: '长度在 3 到 9 个字符', trigger: 'change' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'change' },
          {
            min: 6,
            max: 12,
            message: '长度在 6 到 12 个字符',
            trigger: 'change'
          }
        ]
      }
    }
  },
  methods: {
    reset() {
      this.$refs.form.resetFields()
    },
    // 登录
    login() {
      this.$refs.form.validate(valid => {
        if (!valid) return false
        axios({
          method: 'post',
          url: 'http://localhost:8888/api/private/v1/login',
          data: this.form
        }).then(res => {
          if (res.data.meta.status === 200) {
            this.$message({
              message: '登录成功',
              type: 'success',
              duration: 1000
            })
            localStorage.setItem('token', res.data.data.token)
            localStorage.setItem('whoami', res.data.data.username)
            this.$router.push('/home')
          } else {
            this.$message.error(res.data.meta.msg)
          }
        })
      })
    }
  }
}
</script>

<style lang="less">
.login {
  width: 100%;
  height: 100%;
  position: absolute;
  // background-color: #2d434c;

  .wrapper {
    background-image: url('~@/assets/Vue.png');
    background-position: center;
    overflow: hidden;
    width: 100%;
    height: 100%;
    position: relative;
    filter: blur(5px);
  }
  .el-form {
    opacity: 0.8;
    width: 400px;
    background-color: #fff;
    padding: 75px 40px 15px;
    border-radius: 20px;
    position: absolute;
    z-index: 99;
    top: 40%;
    left: 50%;
    transform: translate(-50%, -50%);
    // filter:blur(0);
    img {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      top: -80px;
      border-radius: 50%;
      border: 10px solid #fff;
      max-width: 130px;
      // filter: blur(10px);
    }
    .el-button + .el-button {
      margin-left: 70px;
    }
  }
}
</style>
