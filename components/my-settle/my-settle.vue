<template>
  <view class="my-settle-container">
    <!-- 全选 -->
    <label class="radio" @click="changeAllState">
      <radio color="#C00000" :checked="isFullcheck" /><text>全选</text>
    </label>
    <!-- 合计 -->
    <view class="amount-box">
      合计:<text class="amount">￥{{checkedGoodsAmount}}</text>
    </view>
    <!-- 结算按钮 -->
    <view class="btn-settle" @click="settlement">
      结算({{checkedCount}})
    </view>
  </view>
</template>

<script>
  import {
    mapGetters,
    mapMutations,
    mapState
  } from 'vuex'
  export default {
    name: "my-settle",
    data() {
      return {
        // 倒计时秒数
        seconds: 3,
        timer: null
      };
    },
    methods: {
      ...mapMutations('m_cart', ['updateAllGoodsState']),
      ...mapMutations('m_user', ['updateRedirectInfo']),
      
      changeAllState() {
        this.updateAllGoodsState(!this.isFullcheck)
      },
      // 用户点击结算按钮
      settlement() {
        if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品！')

        if (!this.addstr) return uni.$showMsg('请选择要收货地址！')

        if (!this.token) {
          // return uni.$showMsg('请先登录！')
          return this.delayNavigate()
        }
        this.payOrder()
      },
      async payOrder() {
        // 创建订单
        const orderInfo = {
          order_price: 0.01,
          consignee_addr: this.attstr,    
          goods: this.cart.filter(x => x.goods_state).map(x => ({
            goods_id: x.goods_id,
            goods_number: x.goods_count,
            goods_price: x.goods_price
          }))
        }
    const { data: res } = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
        if(res.meta.status!== 200) return uni.$showMsg('创建订单失败')
        const orderNumber = res.message.order_number
        // 发起请求创建订单 
        const {data:res2} = await uni.$http.post('/api/public/v1/my/orders/req_unifiedorder',{order_number:orderNumber})
        if(res2.meta.status!== 200) return uni.$showMsg('预付订单生成失败')
        const payInfo = res2.message.pay
        
        // 调用uni.requestPayment()
        const [err,succ] = await uni.requestPayment(payInfo)
        if(err) return uni.$showMsg('订单未支付！')
        // 发送请求判断支付情况
        const {data:res3} = await uni.$http.post('/api/public/v1/my/orders/chkOrder',{order_number:orderNumber})
        if(res3.meta.status !== 200)return uni.$showMsg('订单未支付！')
             uni.showToast ({
               title:'订单支付完成',
               icon:'success'
             })
      },

      // 延时导航至my页面
      // 延迟导航到 my 页面
      delayNavigate() {
        // 重置seconds
        this.seconds = 3
        this.showTips(this.seconds)
        //将定时器的 Id 存储到 timer 中
        this.timer = setInterval(() => {
          this.seconds--
          //  判断秒数是否 <= 0
          if (this.seconds <= 0) {
            // 清除定时器
            clearInterval(this.timer)
            //  跳转到 my 页面
            uni.switchTab({
              url: '/pages/my/my',
              success: () => {
                this.updateRedirectInfo({
                  openType: 'switchTab',
                  from: '/pages/cart/cart'
                })
              }
            })
            // 终止后续代码的运行（当秒数为 0 时，不再展示 toast 提示消息）
            return
          }
          this.showTips(this.seconds)
        }, 1000)
      },
      // 展示倒计时的提示消息
      showTips(n) {
        uni.showToast({
          icon: 'none',
          title: '请登录后再结算！' + n + '秒之后自动跳转到登录页',
          mask: true,
          duration: 1500
        })
      }
    },
    computed: {
      ...mapGetters('m_cart', ['checkedCount', 'total', 'checkedGoodsAmount']),
      ...mapGetters('m_user', ['addstr']),
      ...mapState('m_user', ['token']),
      ...mapState('m_cart', ['cart']),
      isFullcheck() {
        return this.total === this.checkedCount
      }
    }
  }
</script>

<style lang="scss">
  .my-settle-container {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 50px;
    background-color: white;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 14px;
    padding-left: 5px;

    .radio {
      display: flex;
      align-items: center;
    }

    .amount-box {
      .amount {
        color: #C00000;
        font-weight: bold;
      }
    }

    .btn-settle {
      background-color: #C00000;
      height: 50px;
      color: white;
      line-height: 50px;
      padding: 0 10px;
      min-width: 100px;
      text-align: center;
    }
  }
</style>
