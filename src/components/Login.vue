<!-- 登录组件 -->
<template>
  <div class="login_con">
    <div class="login_box">
      <!-- 头像区 -->
      <div class="login_img">
        <img src="../assets/login.png" alt="" />
      </div>

      <!-- 表单域 -->
      <el-form
        ref="loginFormRef"
        :model="loginForm"
        class="login_form"
        :rules="loginRules"
      >
        <!-- 登录框 -->
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="el-icon-user"
            placeholder="请输入用户名"
          ></el-input>
        </el-form-item>

        <!-- 密码框 -->
        <el-form-item prop="password">
          <el-input
            type="password"
            v-model="loginForm.password"
            prefix-icon="el-icon-lock"
            placeholder="请输入密码"
          ></el-input>
        </el-form-item>

        <!-- 按钮 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="loginFormReset">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 登录表单数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    loginFormReset() {
      this.$refs.loginFormRef.resetFields()
    },
    login() {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: resule } = await this.$http.post('login', this.loginForm)
        if (resule.meta.status !== 200)
          return this.$message.error('登陆失败！用户名或密码错误！')
        this.$message.success('登陆成功！')
        window.sessionStorage.setItem('token', resule.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style scoped>
.login_con {
  background-color: #2b4b6b;
  height: 100%;
  width: 100%;
}
.login_box {
  height: 300px;
  width: 450px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.login_img {
  width: 100px;
  height: 100px;
  border: 1px solid #eee;
  padding: 10px;
  border-radius: 50%;
  box-shadow: 0 0 10px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
}

.login_img img {
  height: 100%;
  width: 100%;
  border-radius: 50%;
}

.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0px 40px;
  box-sizing: border-box;
}

.login_form .btns {
  display: flex;
  justify-content: center;
}
</style>
