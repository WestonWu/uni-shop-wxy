<template>
  <view v-if="goods_Info.goods_name" class="goods-detail-container">
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
      <swiper-item v-for="(item, i) in goods_Info.pics" :key="i">
        <image :src="item.pics_big" @click="preview(i)"></image>
      </swiper-item>
    </swiper>
    <!-- 商品信息 -->
    <view class="goods-info-box">
      <!-- price -->
      <view class="price">￥{{goods_Info.goods_price}}</view>
      <!-- 主题区域 -->
      <view class="goods-info-body">
        <!-- name -->
        <view class="goods-name">{{goods_Info.goods_name}}</view>
        <!-- 收藏区域 -->
        <view class="favi">
          <uni-icons type="star" size="18" color="gray"></uni-icons>
          <text>收藏</text>
        </view>

      </view>
      <!-- 运费区域 -->
      <view class="yf">
        快递：免运费
      </view>
    </view>
    <rich-text :nodes="goods_Info.goods_introduce"></rich-text>
    <!-- 商品导航组件 -->
    <view class="goods-nav">
      <uni-goods-nav :fill="true" :options="options" :buttonGroup="buttonGroup" @click="onClick"
        @buttonClick="buttonClick" />
    </view>
  </view>
</template>

<script>
  import {
    mapState,
    mapMutations,
    mapGetters
  } from 'vuex'
  export default {
    computed: {
      ...mapState('m_cart', []),
      ...mapGetters('m_cart', ['total'])
    },
    watch: {
      total: {
        handler(newVal) {
          const findResult = this.options.find(x => x.text === '购物车')
          if (findResult) {
            findResult.info = newVal
          }
        },
        immediate: true
      }
    },
    data() {
      return {
        goods_Info: {},
        options: [{
          icon: 'shop',
          text: '店铺'
        }, {
          icon: 'cart',
          text: '购物车',
          info: 0
        }],
        // 右侧按钮组的配置对象
        buttonGroup: [{
            text: '加入购物车',
            backgroundColor: '#ff0000',
            color: '#fff'
          },
          {
            text: '立即购买',
            backgroundColor: '#ffa200',
            color: '#fff'
          }
        ]
      };
    },
    onLoad(options) {
      const goods_id = options.goods_id
      this.getGoodsDetail(goods_id)
    },
    methods: {
      ...mapMutations('m_cart', ['addToCart']),
      buttonClick(e) {
        if (e.content.text === '加入购物车') {
          // 初始商品信息对象，然后调用addToCart
          // goods_id, goods_name, goods_price, goods_count, goods_small_logo, goods_state
          const goods = {
            goods_id: this.goods_Info.goods_id,
            goods_name: this.goods_Info.goods_name,
            goods_price: this.goods_Info.goods_price,
            goods_count: 1,
            goods_small_logo: this.goods_Info.goods_small_logo,
            goods_state: true,
          }
          this.addToCart(goods)
        }
      },
      async getGoodsDetail(goods_id) {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/detail', {
          goods_id
        })
        if (res.meta.status !== 200) return uni.$showMsg()

        res.message.goods_introduce = res.message.goods_introduce.replace(/<img /g, '<img style="display: block;"')
          .replace(/webp/g, 'jpg')
        this.goods_Info = res.message
      },
      preview(i) {
        uni.previewImage({
          current: i,
          urls: this.goods_Info.pics.map(x => x.pics_big)
        })
      },
      onClick(e) {
        // console.log(e)
        if (e.content.text === '购物车') {
          uni.switchTab({
            url: '../../pages/car/car'
          })
        }
      }
    }
  }
</script>

<style lang="scss">
  swiper {
    height: 750rpx;

    image {
      width: 100%;
      height: 100%;
    }

  }

  .goods-info-box {
    padding: 10px;
    padding-right: 0;

    .price {
      color: #c00000;
      font-size: 18px;
      margin: 10px 0;
    }

    .goods-info-body {
      display: flex;
      justify-content: space-between;

      .goods-name {
        font-size: 13px;
        margin-right: 10px;
      }

      .favi {
        width: 120px;
        font-size: 12px;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        border-left: 1px solid #efefef;
        color: gray;
      }

    }

    .yf {
      font-size: 12px;
      color: gray;
      margin: 10px 0;
    }
  }

  .goods-nav {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
  }

  .goods-detail-container {
    padding-bottom: 50px;
  }
</style>
