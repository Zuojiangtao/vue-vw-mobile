/** * @author zuojt * @date 2020/1/17 * @company MyselfStudy * @description Vue移动端APP自适应登录页面 * Copyright (c)
2020, 左江涛 GROUP. * All rights reserved. */
<template>
  <div class="app">
    <div class="header">
      <span><img src="@/assets/back.png" alt="<" /></span>
      <div class="title ellipsis">移动端APP - 登录</div>
    </div>

    <div class="body">
      <div class="logo">
        <div class="img">vue</div>
        <div class="FIKSlogin">使用手机号登录</div>
        <div class="loginTips">未注册过的手机号将自动创建账号</div>
      </div>
      <div class="signin">
        <div class="form">
          <div class="ipt" id="name">
            <input type="text" placeholder="请输入手机号" v-model.lazy="name" maxlength="11" />
            <button v-show="name" id="deleteName" @click.stop="clearName">
              <img src="@/assets/delete.png" alt="" />
            </button>
          </div>
          <div class="ipt" id="passwordIpt">
            <input type="password" placeholder="短信验证码" v-model="verificateCode" />
            <button @click="getYZM" :disabled="hasGetVerificateCode" :class="hasGetVerificateCode ? 'disabled' : ''">
              {{ hasGetVerificateCode === false ? '获取验证码' : `重新发送(${verifNum})` }}
            </button>
          </div>
        </div>
        <button id="login" @click="login">立即登录/注册</button>
      </div>
    </div>

    <div class="footer">
      <div class="spanLines">
        <span class="lines hairlines"></span>
        <span class="tips">登录即同意该应用获得以下权限</span>
        <span class="lines hairlines"></span>
      </div>
      <ul>
        <li><img src="@/assets/check.png" alt="" /><span>访问您账号的公开信息（用户名、头像等）</span></li>
        <li><img src="@/assets/check.png" alt="" /><span>获取您的账号关联的FIKS设备信息</span></li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Index',
  components: {},
  data() {
    return {
      name: '', // 登录账号名称
      verificateCode: '', // 登录验证码
      hasGetVerificateCode: false, // 点击获取验证码
      verifNum: 60, // 获取验证码倒计时
      timer: null, // 定时器
    }
  },
  created() {},
  mounted() {},
  beforeDestroy() {
    clearInterval(this.timer)
    this.timer = null
  },
  methods: {
    // 清除账号名称
    clearName() {
      this.name = ''
    },
    // 获取验证码
    getYZM() {
      console.log('获取验证码')
      this.hasGetVerificateCode = true
      this.timer = setInterval(() => {
        if (this.verifNum === 0) {
          this.verifNum = 60
          this.hasGetVerificateCode = false
          clearInterval(this.timer)
          this.timer = null
          return
        }
        this.verifNum--
        // console.log(num, this.verifNum)
      }, 1000)
    },
    // 登录
    login() {
      console.log('登录')
    },
  },
}
</script>

<style lang="scss" scoped>
@import 'index.scss';
</style>
