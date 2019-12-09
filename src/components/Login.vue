<template>
  <div class="login-container">
    <div class="login-box">
      <div class="avatar-box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!-- 表单登录区域 -->
      <el-form
        label-width="60px"
        class="login-form"
        :model="loginForm"
        :rules="loginFormRules"
        ref="loginFormRef"
        >
        <el-form-item label="账号" prop="username">
          <el-input prefix-icon="iconfont icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type="password" prefix-icon="iconfont icon-3702mima" v-model="loginForm.password"></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>

export default {
  data () {
    return {
      loginForm : {
        username: '',
        password: ''
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm() {
      console.log(this)
      this.$refs.loginFormRef.resetFields() 
    },
    login() {
      this.$refs.loginFormRef.validate(valid => {
        console.log(valid)
        if (!valid) return
        this.$http.post('/login', this.loginForm).then(res => {
          console.log(res)
          if (res.data.meta.status !== 200) {
            console.log('登录失败')
            this.$message.error('登录失败');
          } else {
           this.$message.success('登录成功！')
          }

          window.sessionStorage.setItem("token", res.data.data.token)
          this.$router.push('/home')
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .login-container {
    background: #2b4b;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .login-box {
    height: 300px;
    width: 450px;
    background: #fff;
    border-radius: 10px;
    position: relative;

    .avatar-box {
      height: 130px;
      width: 130px;
      background: #2b4b;
      border-radius: 50%;
      border: 1px solid red;
      padding: 10px;
      box-shadow: 0 0 10px #ddd;
      position: absolute;
      left: 50%;
      transform: translate(-50%, -50%);

      img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background: #eee;
      }
    }

    .login-form {
      position: absolute;
      bottom: 0;
      width: 100%;
      padding: 0 40px;
      box-sizing: border-box;
    }

    .btns {
      display: flex;
      justify-content: flex-end;
    }
  }
</style>