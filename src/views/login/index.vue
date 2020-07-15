<template>
  <div class="login-wrap">
    <div class="login-form-wrap">
      <div class="login-head">
        <img src=".login_index.png" alt="黑马头条">
      </div>
      <div class="login-form">
        <!-- 
          表单验证：
        rules 配置验证规则
        将需要验证的字段通过prop 属性配置到 el-form-item 组件上
        ref 获取表单组件 可以手动调用表单组件的验证方法 
        -->
        <el-form ref="form" :model="form" :rules="rules" ref="ruleForm">
          <el-form-item prop="mobile">
            <el-input v-model="form.mobile" placeholder="手机号"></el-input>
          </el-form-item>
          <el-form-item prop="code">
            <!-- 支持栅格布局一共是24行 -->
            <el-col ;span="10">
              <el-input v-model="form.code" placeholder="验证码"></el-input>
            </el-col>
            <el-col ;span="10" ;offset="2">
              <el-button @click="handleSendCode" >获取验证码</el-button>
            </el-col>
          </el-form-item>
          <el-form-item prop="agree">
            <el-checkbox v-model="form.agree"></el-checkbox>
            <span>我已阅读并同意<a href="#">用户协议</a>和<a href="#">隐私条款</a></span>
          </el-form-item>
          <el-form-item>
            <!-- 给组件加class会作用到它的根元素 -->
            <el-button 
              class="btn-login" 
              type="primary" 
              @click="handLogin"
              :loading="loginLoading"
            >登录</el-button>
          </el-form-item>
        </el-form> 
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import '@/vendor/gt'

export default {
  name: 'Applogin',
  data () {
    return {
      form: { // 表单数据
          mobile: '13121840224',
          code: '' ,// 是否同意用户协议
          agree： ''
      }，
      loginLoading: false // 登录按钮的loading状态
      rules: { // 表单验证规则
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { min: 11, max: 11, message: '长度必须为11个字符', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' },
          { min: 6, max: 6, message: '长度必须为6个字符', trigger: 'blur' }
          ]
        agree: [
          { required: true, message: '请同意用户协议', trigger: 'change' },
          { pattern: /true/, message: '请同意用户协议', trigger: 'change' }
        ]
      },
      captchaObj: null   // 通过initGeetest 得到的极验验证码对象
    }
  },

  methods: {
    handLogin () {
      表单组件有一个方法 validate 可以用于获取当前表单的验证状态
      this.$refs['ruleForm'].validate(valid => {
        if (!valid) {
          return
        }

        // 表单验证通过，提交登录
        this.submitLogin()
      })
    },

    submitLogin () {
      this.loading = true
      axios({
        method: 'POST',
        url: http://meiyoujutidejiekousuibianxieba,
        data: this.form
      }).then(res => {// >=200 && < 400 的状态都会进入这里
        // Element 提供的 message 消息组件
        this.$message({
          message: '恭喜你这是一条成功消息'，
          type: 'success'
        })

        this.loginLoading = false
        
        // 建议路由跳转都用name去跳转，路由传参非常方便
        this.$router.push({
          name:'home'
        })
      }).catch(err => {// >= 400 de HTTP 状态码都会进入catch中
        if (err.response.statu === 400) {
          this.$message.error('登陆失败，手机或验证码错误')
        }
         this.loginLoading = false   
      })
    },

    handleSendCode () {
      // 校验手机号是否有效 先拿到表单组件，然后在调用对单个表单的Validate验证的方法 this.$refs['ruleForm']是操作表单DOM元素
      this.$refs['ruleForm'].validateField('mobile', errorMessage => {
      if (errorMessage.length > 0) {
        return
        }
        // 判手机号有效 初始化验证码插件
        this.showGeetest()
      })
    },

    showGeetest () {
      const { mobile } = this.form
      
      if (this.captchaObj) {
        return this.captchaObj.verify()
      }

      axios({
        method:'GET'
        url:'http://meiyoujutidejiekousuibianxieba/${mobile}'
      }).then(res => {
        // console.log(res.data)
        const data = res.data.data
        initGeetest({
            // 以下配置参数来自服务端 SDK
            gt: data.gt,
            challenge: data.challenge,
            offline: !data.success,
            new_captcha: data.new_captcha,
            product: 'bind'
        }, function (captchaObj) => {
          this.captchaObj = captchaObj
            // 这里可以调用验证实例 captchaObj 的实例方法
            captchaObj.onReady(function () {
              captchaObj.verify(); //显示验证码
          }).onSuccess(function () {
            const {
              geetest_challenge: challenge,
              geetest_seccode: seccode,
              geetest_validate: validate} = 
            captchaObj.getValidate()
            axios({
              method:'GET'
              url:'http://meiyoujutidejiekousuibianxieba/${mobile}'，
              params : {
                challenge,
                seccode,
                validate
              }
            }).then(res => {
              console.log(res.data)
            })
          })
        })
      }
    })
  }
}
</script>

<style lang="less" scoped>
.login-wrap {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ccc;
  .login-head{
    display: flex;
    justify-content: center;
    margin-bottom: 10px;
    img{
      width: 200px;
    }
  }
  .login-form-wrap{
    background-color: #fff;
    padding: 50px;
    border-radius: 10px;
    .btn-login {
      width: 100%;
    }
  }
}
</style>