<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
          <img src="../assets/logo.png" alt="">
      </div>
      <el-form ref="LoginFormRef" 
      :model="loginForm" 
      label-width="0px"
      :rules='loginFormRules'
      class="login_form">
        <el-form-item  prop="username">
          <el-input v-model="loginForm.username"></el-input>
        </el-form-item>
        <el-form-item  prop="password" type='password'>
          <el-input v-model="loginForm.password"></el-input>
        </el-form-item>
        <el-form-item class="btns">
            <el-button type="primary" @click="login">登录</el-button>
            <el-button type='info' @click='resetLoginForm'>重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      loginForm: {
        username: 'admin',
        password: '123456',
      },
      loginFormRules:{
        username:[
           { required: true, message: '请输入登录名称', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password:[
          {required:true,message:'请输入密码',trigger:'blur'},
          {min:6,max:15,message:'长度在6到15个字符',trigger:'blur'}
        ]
      }
    }
  },
  methods: {
      login(){
          this.$refs.LoginFormRef.validate (async valid=>{
              if(!valid) return
              const {data:res}= await this.$http.post('login',this.loginForm)
              if (res.meta.status!== 200) return this.$message.error('登录失败');
              this.$message.success('登录成功')
            window.sessionStorage.setItem('token',res.data.token)

            this.$router.push('/home')
    
          })

      },
      resetLoginForm(){
          this.$refs.LoginFormRef.resetFields()
      },
  },
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box{
    width: 450px;
    height: 300px;
    background: white;
    position: absolute;
    top:50%;
    left: 50%;
    transform: translate(-50%,-50%);
    border-radius: 3px;
    .avatar_box {
        width: 130px;
        height: 130px;
        position: absolute;
        left: 50%;
        transform: translate(-50%,-50%);
        border-radius: 50%;
        border:1px white solid;
        background-color: #fff;
        padding:10px;
        img {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background-color: #eee;
        }
    }
}
.login_form{
    position: absolute;
    bottom: 0px;
    width: 100%;
    padding: 0 20px;//上下左右
    box-sizing: border-box;
}
.btns{
    display: flex;
    justify-content: flex-end;//按钮靠右
}
</style>

