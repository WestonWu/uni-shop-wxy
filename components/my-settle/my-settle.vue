<template>
  <view class="my-settle-container">
    <!-- 全选 -->
    <label class="radio" @click="changeAllState">
      <radio color="#c00000" :checked="isFullChecked" /><text>全选</text>
    </label>
    <!-- 合计 -->
    <view class="amount-box">
      合计：<text class="amount">￥{{checkedGoodsAmount}}</text>
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
    computed: {
      ...mapGetters('m_cart', ['checkedCount', 'total', 'checkedGoodsAmount']),
      ...mapGetters('m_user', ['addstr']),
      ...mapState('m_user', ['token']),
      ...mapState('m_cart', ['cart']),
      isFullChecked() {
        return this.checkedCount === this.total
      }
    },
    methods: {
      ...mapMutations('m_cart', ['updateAllGoodsState']),
      ...mapMutations('m_user', ['updateReDirectInfo']),
      changeAllState() {
        // console.log(!this.isFullChecked)
        this.updateAllGoodsState(!this.isFullChecked)
      },
      settlement() {
        if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品！')
        if (!this.addstr) return uni.$showMsg('请选择收货地址！')
        // if(!this.token) return uni.$showMsg('请先登录！')
        if (!this.token) return this.delayNavigate()

        this.payOrder()
      },
      async payOrder() {
        const orderInfo = {
          // order_price: this.checkedGoodsAmount,
          order_price: 0.01,
          consignee_addr: this.addstr,
          goods: this.cart.filter(x => x.goods_state).map(x => ({
            goods_id: x.goods_id,
            goods_number: x.goods_count,
            goods_price: x.goods_price
          }))
        }

        const {
          data: res
        } = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
        // console.log(res)
        // if(res.meta.status !== 200) return uni.$showMsg('创建订单失败！')


        const orderNumber = 'GD20180504000000000045'
        // console.log(orderNumber)
        const {
          data: res2
        } = await uni.$http.post('/api/public/v1/my/orders/req_unifiedorder', {
          order_number: orderNumber
        })

        // if(res.meta.status !== 200) return uni.$showMsg('预付订单生成失败！')
        const payInfo = {
          "timeStamp": "1525681145",
          "nonceStr": "BkPggorBXZwPGXe3",
          "package": "prepay_id=wx071619042918087bb4c1d3d72999385683",
          "signType": "MD5",
          "paySign": "D1642DEEF1663C8012EDEB9297E1D516"
        }
        // console.log(payInfo)
        const [err, succ] = await uni.requestPayment(payInfo)        
        console.log(err + '...' + succ)
        if(err) return uni.$showMsg('订单未支付！')
        
        const {data: res3} = await uni.$http.post('/api/public/v1/my/orders/chkOrder', {order_number: orderNumber})
        
        // if(res3.meta.status !== 200) return uni.$showMsg('订单未支付！')
        uni.showToast({
          title: '订单支付完成',
          icon: 'success'
        })
      },
      // 展示倒计时
      showTips(n) {
        uni.showToast({
          icon: 'none',
          title: '请登录后再结算！' + n + '秒后自动跳转到登录页',
          mask: true,
          duration: 1500
        })
      },
      // 延时导航到my页面
      delayNavigate() {
        this.seconds = 3
        this.showTips(this.seconds)

        this.timer = setInterval(() => {
          this.seconds--
          if (this.seconds <= 0) {
            clearInterval(this.timer)
            uni.switchTab({
              url: '../../pages/my/my',
              success: () => {
                this.updateReDirectInfo({
                  openType: 'switchTab',
                  from: '../../pages/car/car'
                })
              }
            })
            return
          }
          this.showTips(this.seconds)
        }, 1000)
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
        color: #c00000;
        font-weight: bold;
      }
    }

    .btn-settle {
      background-color: #c00000;
      height: 50px;
      color: white;
      line-height: 50px;
      padding: 0 10px;
      min-width: 100px;
      text-align: center;
    }
  }
</style>
