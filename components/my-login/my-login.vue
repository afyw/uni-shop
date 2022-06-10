<template>
  <view class="login-container">
    <uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
    <button type="primary" class="btn-login" @click="getUserProfile">一键登录 </button>
    <text class="tips-text">登录后尽享更多权益</text>
  </view>
</template>

<script>
  import {mapMutations,mapState} from 'vuex'
  export default {
    name: "my-login",
    data() {
      return {

      };
    },
    computed:{
      ...mapState('m_user',['redirectInfo'])
    },
    methods: {
      ...mapMutations('m_user',['updateUserInfo','updateToken','updateRedictInfo']),
      // 用户授权之后，获取用户的基本信息
      getUserProfile() {
        uni.getUserProfile({
          desc: '你的授权信息',
          success: (res) => {
            // 将信息存到 vuex 中
            this.updateUserInfo(res.userInfo)
            this.getToken(res)
          },
          fail: (res) => {
            return uni.$showMsg('您取消了登录授权')
          }
        })
      },
      async getToken(info){
        // 获取code对应的值
        const [err,res] = await uni.login().catch(err => err)
        if(err || res.errMsg !== 'login:ok')return uni.$showMsg('登录失败！')
        // 准备请求的参数
        const query = {
          code:res.code,
          encryptedData: info,
          iv: info.iv,
          rawData:info.rawData,
          signature: info.signature
        }
       const {data:loginResult} = await uni.$http.post('/api/public/v1/users/wxlogin',query)
       // 此处需要小程序开发者设置权限 status!== 200
       if(loginResult.meta.status == 200) return uni.$showMsg('登录失败！')
       // 直接把token保存到vuex中 此处把code传入本地保存
       this.updateToken("Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOjEyLCJpYXQiOjE1MjU0MDIyMjMsImV4cCI6MTUyNTQ4ODYyM30.g-4GtEQNPwT_Xs0Pq7Lrco_9DfHQQsBiOKZerkO-O-o")
       this.navigateBack()
      },
      navigateBack(){
        if(this.redirectInfo && this.redirectInfo.openType=== 'switchTab'){
          uni.switchTab({
            url:this.redirectInfo.from,
            commplete:() =>{
              this.updateRedictInfo(null)
            }
          })
        }
      }
    }
  }
</script>

<style lang="scss">
  .login-container {
    height: 750rpx;
    background-color: #F8F8F8;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    overflow: hidden;

    &::after {
      content: ' ';
      display: block;
      width: 100%;
      height: 40px;
      background-color: white;
      position: absolute;
      bottom: 0;
      left: 0;
      border-radius: 100%;
      transform: translateY(50%);
    }

    .btn-login {
      width: 90%;
      border-radius: 100px;
      margin: 15px 0;
      background-color: #C00000;
    }

    .tips-text {
      font-size: 12px;
      color: gray;
    }
  }
</style>
