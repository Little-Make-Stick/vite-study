
 
<template>
  <div id="login" class="login" @keydown.enter="handleSubmit">
    <div class="login-con">
      <Card :bordered="false" style="width: 350px;height: 380px">
        <div class="form-con">
          <form ref="loginForm" :model="form" :rules="rules" :label-width="90" inline>
            <div class="form-item" label="帐号" prop="username">
              <label for="username">帐号</label>
              <Input v-model="form.username" placeholder="请输入帐号" ref="input" clearable style="width: 200px" />
            </div>
            <div class="form-item" label="密码" prop="password">
              <label for="password">密码</label>
              <Input v-model="form.password" placeholder="请输入密码" clearable style="width: 200px" />
            </div>
            <div class="form-item" label="验证码" prop="verificationCode">
              <label for="verificationCode">验证码</label>
              <Input v-model="form.verificationCode" placeholder="请输入验证码" clearable style="width: 200px" />
              <div @click="refreshCode" style="margin-top: 20px">
                <!--验证码组件-->
                <v-identify :identifyCode="identifyCode"></v-identify>
              </div>
            </div>
            <div style="text-align: center">
              <button @click="handleSubmit" type="primary" style="margin-right: 20px">登录</button>
            </div>
          </form>
        </div>
      </Card>
    </div>
  </div>
</template>
 
<script>
import VIdentify from "./components/VIdentify.vue";

export default {
  components: { VIdentify },
  name: "login",
  data() {
    return {
      form: {},
      formValidate: {},
      rules: {
        username: [
          { required: true, message: "登录用户名不能为空", trigger: "blur" },
        ],
        password: [
          { required: true, message: "登录密码不能为空", trigger: "blur" },
        ],
        verificationCode: [
          { required: true, message: "验证码不能为空", trigger: "blur" },
        ],
      },
      ruleValidate: {
        username: [
          { required: true, message: "登录用户名不能为空", trigger: "blur" },
        ],
        password: [
          { required: true, message: "登录密码不能为空", trigger: "blur" },
        ],
        mobile: [
          { required: true, message: "手机号不能为空", trigger: "blur" },
        ],
        header: [
          { required: true, message: "短信验证码不能为空", trigger: "blur" },
        ],
      },
      // 是否禁用按钮
      codeDisabled: false,
      // 倒计时秒数
      countdown: 60,
      // 按钮上的文字
      codeMsg: "获取验证码",
      // 定时器
      timer: null,
      identifyCode: "",
      identifyCodes: "1234567890abcdefjhijklinopqrsduvwxyz",
    };
  },
  methods: {
    // 刷新验证码
    refreshCode() {
      this.identifyCode = "";
      this.makeCode(this.identifyCodes, 4);
    },
    makeCode(o, l) {
      for (let i = 0; i < l; i++) {
        this.identifyCode +=
          this.identifyCodes[this.randomNum(0, this.identifyCodes.length)];
      }
    },
    randomNum(min, max) {
      return Math.floor(Math.random() * (max - min) + min);
    },
    // 获取短信验证码
    getCode() {
      // 验证码60秒倒计时
      if (!this.timer) {
        this.getValidStr();
        this.timer = setInterval(this.getValidStr, 1000);
      }
      apiRequest(this, getCode(this.form.mobile), (response) => {});
    },
    getValidStr() {
      if (this.countdown > 0 && this.countdown <= 60) {
        this.countdown--;
        if (this.countdown !== 0) {
          this.codeMsg = "重新发送(" + this.countdown + ")";
          this.codeDisabled = true;
        } else {
          clearInterval(this.timer);
          this.codeMsg = "获取验证码";
          this.countdown = 60;
          this.timer = null;
          this.codeDisabled = false;
        }
      }
    },
    handleSubmit() {
      this.$refs.loginForm.validate((valid) => {
        if (valid) {
          //登录密码做MD5加密
          let password = this.$copyto.md5(this.form.password);
          //登录接口请求
          apiRequest(this, login(this.form.username, password), (response) => {
            this.$store.commit("setUserInfo", response.data);
            Cookies.set("user", this.form.username);
            Cookies.set("userId", response.data.id);
            localStorage.sessionId = response.sessionId;
            this.$store.commit("setAvator", "");
            if (this.form.userName === "admin") {
              Cookies.set("access", 0);
            } else {
              Cookies.set("access", 1);
            }
            this.$router.push({ name: "home_index" });
          });
        }
      });
    },
    register() {},
  },
  mounted() {
    // 初始化验证码
    this.identifyCode = "";
    this.makeCode(this.identifyCodes, 4);
  },
};
</script>
 
<style lang="less">
</style>