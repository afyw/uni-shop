<template>
  <view>
    <!-- 搜索组件 -->
    <view class="search-box">
      <my-search @myclick="gotoSearch"></my-search>
    </view>
     <!-- 轮播图区域 -->
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" circular="true">
      <swiper-item v-for="(item,index) in swiperList" :key="index">
        <navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id='+ item.goods_id" >
          <image :src="item.image_src" class="nav-img"></image>
        </navigator>
      </swiper-item>
    </swiper>
    
    <!-- 分类导航区域 -->
    <view class="nav-list">
      <view class="nav-item" v-for="(item,index) in navList" :key="index" @click="navClickHandler(item)">
        <img :src="item.image_src"  class="nav-img">
      </view>
    </view>
    
    <!-- 楼层区域 -->
    <view class="floor-list">
      <!-- 每个楼层的item项目 -->
      <view class="floor-item" v-for="(item,index) in floorList" :key="index">
        <!-- 楼层标题 -->
        <image :src="item.floor_title.image_src" class="floor-title"></image>
      
      <!-- 楼层图片区域 -->
      <view class="floor-img-box">
        <!-- 左侧大图片 -->
        <navigator class="left-img-box" :url="item.product_list[0].url">
            <image :src="item.product_list[0].image_src"  :style="{width:item.product_list[0].image_width + 'rpx'}" mode="widthFix" ></image>
          </navigator>
          <!-- 右侧的小盒子 -->
          <view class="right-img-box">
            <navigator class="right-img-item" v-for="(item2,index2) in item.product_list" :key="index2" v-if="index2 !== 0" :url="item2.url"> 
              <image :src="item2.image_src" :style="{width:item2.image_width + 'rpx'}" mode="widthFix"></image>
            </navigator>
          </view>
        </view>
      </view>
      </view>
    </view>
    
</template>

<script>
  export default {
    data() {
      return {
        // 轮播图数据列表
        swiperList:[],
        // 分类导航的数据列表
        navList:[],
        // 楼层数据
        floorList:[]
      };
    },
    onLoad(){
      // 获取轮播图数据
      this.getSwiperList()
      this.getNavList()
      this.getFloorList()
    },
    methods:{
      async getSwiperList(){
        const {data:res} = await uni.$http.get('/api/public/v1/home/swiperdata')
        if(res.meta.status !== 200) return uni.$showMsg()
        this.swiperList = res.message
      },
      async getNavList(){
        const {data:res} = await uni.$http.get('/api/public/v1/home/catitems')
        if(res.meta.status !== 200) return uni.$showMsg()
        this.navList = res.message 
      },
      navClickHandler(item){
        if(item.name==='分类'){
          uni.switchTab({
            url:'/pages/cate/cate'
          })
        }
      },
      async getFloorList(){
        const {data:res} = await uni.$http.get('/api/public/v1/home/floordata')
        if(res.meta.status !== 200) return uni.$showMsg()
        
        // 对数据进行处理
        res.message.forEach(floor =>{
          floor.product_list.forEach(prod=>{
            prod.url = '/subpkg/goods_list/goods_list?' +prod.navigator_url.split('?')[1]
          })
        })
        this.floorList = res.message
      },
      gotoSearch(){
        uni.navigateTo({
          url:"/subpkg/search/search"
        })
      }
    }
  }
</script>

<style lang="scss">
  swiper{
    height: 330rpx;
    .swiper-item,
    image{
      width: 100%;
      height:100%;
    }
  }
  .nav-list{
    display:flex;
    justify-content: space-around;
    margin: 15px 0;
    
    .nav-img{
      width: 128rpx;
      height: 140rpx;
    }
  }
  
  .floor-title{
    height: 60rpx;
    width: 100%;
  }
  .right-img-box{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
  }
  .floor-img-box{
    display: flex;
    padding-left: 10rpx;
  }
  
    
  .search-box{
    position: sticky;
    top:0;
    z-index: 999;
  }
</style>
