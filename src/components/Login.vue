<template>
    <div class="login_container" @keyup.enter="login">
      <div class="login_box">
        <!-- 头像区域-->
        <!--登陆表单区域 -->
        <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" class="login_form">
          <!--用户名-->
          <el-form-item prop="username">
            <el-input v-model="loginForm.username" prefix-icon="iconfont icon-icon_people_fill"></el-input>
          </el-form-item>
          <!--密码-->
          <el-form-item prop="password">
            <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input>
          </el-form-item>
          <!--按钮区域-->
          <el-form-item class="btns">
            <el-button type="primary" @click="login">登陆</el-button>
            <el-button type="info" @click="resetLoginForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
</template>
<script>
export default {
  name: 'Login',
  data () {
    return {
      loginForm: {
        username: '',
        password: ''
      },
      // 表单验证规则
      loginFormRules: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 5, max: 11, message: '用户名长度在 5 到 11 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 16, message: '密码长度在 6 到 16 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 重置表单
    resetLoginForm () {
      console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    // 表单提交预验证
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message({ message: '用户名密码输入错误', type: 'error' })
        this.$message({ message: '登陆成功', type: 'success' })
        // 1：将登陆成功之后的token， 保存到客户端sessionStorage中
        // 1.1：项目中除了登录之外的其他api接口，必须在登录之后才能访问
        // 1.2：token直营在当前网站打开期间生效，所以将token保存在sessionStorage（临时储存）中而不是localStora（永久储存）中
        window.sessionStorage.setItem('token', res.data.token)
        // 通过编程式导航跳转到后台主页，路由地址是home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .login_container{
    background-color: #99ffff;
    height: 100%;
  }
  .login_box{
    width: 450px;
    height: 300px;
    background-color: #ffff99;
    border-right: 3px;
    border-radius: 30px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform:translate(-50%, -50%);
  }
  .login_form{
    position: absolute;
    bottom: 30px;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
  }
  .btns {
    display: flex;
    justify-content: flex-end;
  }
</style>
