<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!-- 表单区域 -->
      <el-form ref="loginFormRef" :model='loginForm' :rules='loginFormRules'  class="login_form">
        <!-- 用户名 -->
        <el-form-item prop="username">
           <el-input prefix-icon="el-icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop='password'>
           <el-input prefix-icon="el-icon-lock" v-model="loginForm.password" show-password></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item  class="btns">
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
      // 这是登录表单的数据绑定对象
      loginForm: {
        username: '',
        password: ''

      },
      // 这是表单验证对象
      loginFormRules: {
        // 验证用户名合法性
        username: [
          { required: true, message: '请输入用户名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12 个字符', trigger: 'blur' }
        ]

      }
    }
  },
  methods: {
    // 点击重置按钮，重置登录表单
    resetLoginForm: function () {
      // console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    login: function () {
      this.$refs.loginFormRef.validate(async valid => {
        // console.log(valid)

        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) {
          return this.$message.error('登录失败')
        }
        this.$message.success('登录成功')
        // 1.将登录成功的token，保存到客户端的sessionStorage中
        // 1.1项目中除了登录之外的其他API接口，必须在登录之后才能访问
        // 1.2token 只应在当前网站打开期间生效，所以将token保存在sessionStorage中
        window.sessionStorage.setItem('token', res.data.token)
        // 2.通过编程式导航跳转到后台主页，路由地址是 /home
        this.$router.push('/home')
      })
    }
  }

}
</script>

<style lang="less" scoped>
.login_container{
  position: relative;
  height: 100%;
  background-color: #2b4b6b;

  .login_box{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;

    .avatar_box{
      position: absolute;
      left: 50%;
      transform: translate(-50%,-50%);
      width: 130px;
      height: 130px;
      border: 1px solid #eee;
      border-radius: 50%;
      overflow: hidden;
      padding: 10px;
      box-shadow: 0 0 10px #ddd;
      background-color: #fff;

      img{
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: #eee;
      }
    }
  }
}
.login_form{
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
