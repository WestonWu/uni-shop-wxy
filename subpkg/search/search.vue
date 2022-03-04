<template>
  <view>
    <view class="search-box">
      <uni-search-bar :radius="100" @input="input" cancelButton="none" placeholder="请输入搜索内容" focus="true"></uni-search-bar>
    </view>
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetails(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="right" size="16"></uni-icons>
      </view>
    </view>
    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clean"></uni-icons>
      </view>
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in histories" :key="i" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        timer: null,
        kw: '',
        searchResults: [],
        historyList: []
      };
    },
    onLoad() {
      this.historyList = JSON.parse(uni.getStorageSync('kw') || [])
    },
    methods: {
      input(e) {
        clearTimeout(this.timer)
        this.timer = setTimeout(() => {
          this.kw = e.valueOf()
          this.getSearchList()
        }, 500)
      },
      async getSearchList() {
        if (this.kw.length === 0) {
          this.searchResults = []
          return
        }
        
        const {data: res} = await uni.$http.get('/api/public/v1/goods/qsearch', {query: this.kw})
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message
        this.saveSearchHistory()
      },
      gotoDetails(item) {
        uni.navigateTo({
          url: '../goods_Details/goods_Details?goods_id=' + item.goods_id
        })
      },
      saveSearchHistory() {
        // this.historyList.push(this.kw)
        const set = new Set(this.historyList)
        set.delete(this.kw)
        set.add(this.kw)
        this.historyList = Array.from(set)
        //对搜索历史数据进行持久化存储
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
      },
      clean() {
        this.historyList = []
        uni.setStorageSync('kw', '[]')
      },
      gotoGoodsList(item){
        uni.navigateTo({
          url: '../goods_List/goods_List?query=' + item
        })
      }
    },
    computed: {
      histories () {
        return [...this.historyList].reverse()
      }
    }
  }
</script>

<style lang="scss">
  .search-box{
    position: sticky;
    top: 0;
    z-index: 999;
  }
  .sugg-list{
    padding: 0 5px;
    .sugg-item{
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      .goods-name{
        //文本不允许换行
        white-space: nowrap;
        //溢出部分隐藏
        overflow: hidden;
        //溢出部分用...替代
        text-overflow: ellipsis;
  }}}
  .history-box{
    padding: 0 5px;
    .history-title{
      display: flex;
      justify-content: space-between;
      height: 40px;
      align-items: center;
      font-size: 13px;
      border-bottom: 1px solid #efefef;
    }
    .history-list{
      display: flex;
      flex-wrap: wrap;
      .uni-tag {
        margin-top: 5px;
        margin-right: 5px;
      }
    }
  }
</style>
