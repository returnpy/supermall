<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control class="tab-control" :titles="['流行', '新款', '精选']" @tabClick="tabClick" ref="tabControl1" v-show="isTabFixed"></tab-control>
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll" :pull-up-load="true" @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"></recommend-view>
      <feature-view></feature-view>
      <tab-control :titles="['流行', '新款', '精选']" @tabClick="tabClick" ref="tabControl2"></tab-control>
      <goods-list :goods="showGoods"></goods-list>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
import HomeSwiper from "./childComps/HomeSwiper"
import RecommendView from "./childComps/RecommendView"
import FeatureView from "./childComps/FeatureView"

import NavBar from "components/common/navbar/NavBar"
import Scroll from "components/common/scroll/Scroll"
import TabControl from "components/content/tabControl/TabControl"
import GoodsList from "components/content/goods/GoodsList"
import BackTop from "components/content/backTop/BackTop"

import { getHomeMultidata, getHomeGoods } from "../../network/home"
import { debounce } from "common/utils"

export default {
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] }
      },
      categories: ['pop', 'new', 'sell'],
      currentCate: 0,
      isShowBackTop: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0
    };
  },
  components: {
    HomeSwiper,
    RecommendView,
    FeatureView,
    NavBar,
    Scroll,
    TabControl,
    GoodsList,
    BackTop
  },
  created() {
    // 请求home头数据
    this.getHomeMultidata();

    // 请求商品数据
    this.getHomeGoods('pop');
    this.getHomeGoods('new');
    this.getHomeGoods('sell');

  },
  mounted() {
    
    // 监听item中图片加载完成
    const refresh = debounce(this.$refs.scroll.refresh, 100)

    this.$bus.$on('itemImageLoad', () => {
      refresh()
    })
  },
  methods: {

    // 事件监听相关方法
    tabClick (index) {
      
      this.currentCate = index
      this.$refs.tabControl1.currentIndex = index
      this.$refs.tabControl2.currentIndex = index
    },
    backClick () {
      this.$refs.scroll.scrollTo(0, 0, 500)
    },
    contentScroll (position) {
      // 是否显示 BackTop
      this.isShowBackTop = -position.y > 1000

      // 决定 tabControl 是否吸顶
      this.isTabFixed = -position.y > this.tabOffsetTop
    },
    loadMore () {
      this.getHomeGoods(this.categories[this.currentCate])
    },
    swiperImageLoad () {
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop

    },

    // 网络请求相关方法
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },

    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page++

        this.$refs.scroll.finishPullUp()
      });
    }
  },
  computed: {
    showGoods () {
      return this.goods[this.categories[this.currentCate]].list
    }
  },
  activated() {
    this.$refs.scroll.scrollTop(0, -this.saveY, 0)
    this.$refs.scroll.refresh()
  },
  deactivated() {
    this.saveY = this.$refs.scroll.getScrollY()
  },
};
</script>

<style scoped>
#home {
  height: 100vh;
  position: relative;
}

.home-nav {
  background-color: var(--color-tint);
  color: #fff;

}

.tab-control {
  position: relative;
  z-index: 9;
  background: #fff;
}

.content {
  height: calc(100% - 93px);
  overflow: hidden;

  position: absolute;
  top: 44px;
  bottom: 40px;
  left: 0;
  right: 0;

}

</style>