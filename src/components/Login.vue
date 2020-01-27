<template>
  <div class="login_container">
    <div class="login_box">
      <div id="login_logo"><img src="../assets/logo.png" alt="" /></div>
      <div id="login_main">
        <el-form
          :model="loginForm"
          ref="loginFormRef"
          :rules="loginFormRules"
          label-width="0px"
          class="login_form"
        >
          <!--用户名-->
          <el-form-item prop="username">
            <el-input
              placeholder="请输入用户名"
              prefix-icon="el-icon-user"
              v-model="loginForm.username"
              v-focus
            ></el-input>
          </el-form-item>
          <!--密码-->
          <el-form-item prop="password">
            <el-input
              type="password"
              placeholder="请输入密码"
              prefix-icon="el-icon-lock"
              v-model="loginForm.password"
            ></el-input>
          </el-form-item>
          <el-form-item class="btns">
            <el-button type="primary" @click="login">登录</el-button>
            <el-button @click="resetLoginForm">重置</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 数据校验规则
      loginFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, message: '长度不能小于6个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm() {
      this.$refs.loginFormRef.resetFields()
    },
    login() {
      this.$refs.loginFormRef.validate(valid => {
        if (!valid) return
        this.$http.post('login', this.loginForm).then(result => {
          let data = result.data
          // eslint-disable-next-line eqeqeq
          if (data.meta.status == 200) {
            this.$message.success('登陆成功')
            window.sessionStorage.setItem('token', data.data.token)
            window.sessionStorage.setItem('username', data.data.username)
            this.$router.push('/home')
          } else {
            this.$message.error(data.meta.msg)
          }
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
  width: 100%;
}

.login_box {
  width: 450px;
  height: 304px;
  background-color: #fff;
  border-radius: 4px;
  position: absolute;
  top: 35%;
  left: calc(50% - 450px / 2);
}

#login_logo {
  width: 130px;
  height: 130px;
  border-radius: 50%;
  border: 1px solid #eee;
  padding: 10px;
  position: absolute;
  box-shadow: 0 0 10px #eee;
  background: #fff;
  left: 50%;
  transform: translate(-50%, -50%);
  img {
    width: 100%;
    height: 100%;
    border: 1px solid #fff;
    border-radius: 50%;
    background-color: #eee;
  }
}

.btns {
  display: flex;
  justify-content: flex-end;
}

#login_main {
  margin-top: 100px;
  margin-left: 30px;
  margin-right: 30px;
}
</style>
