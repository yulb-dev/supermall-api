<template>
  <div class="body">
    <main-nav-bar>
      <div slot="left">
        <i class="iconfont icon-search"></i>
        <input
          type="text"
          placeholder="裂变6"
          @keydown.enter="search"
          v-model="char"
        />
      </div>
      <div slot="right">
        <i class="iconfont icon-xiangji1"></i>
        <p>拍照搜</p>
      </div>
    </main-nav-bar>
    <scroll
      ref="scroll"
      :hideShowBackTop="true"
      @goTop="goTop"
      @pullingUp="pullingUp"
    >
      <swiper2
        :imglist="imglist"
        @ctoItemDetails="ctoItemDetails"
        ref="swiper"
      />
      <main-recomm :recoImgList="recoImgList" />
      <main-activity />
      <main-switch @cswitch="cswitch" />
      <main-goods-list :goodsList="this.goods[sle].list" />
    </scroll>
    <back-top @click.native="backlick" v-show="isShow" />
    <main-info
      v-show="infoIsShow"
      :swiperItem="swiperItem"
      @shutInfo="shutInfo"
    ></main-info>
  </div>
</template>
<script>
import MainSwitch from "../../componets/content/MainSwitch/MainSwitch";
import MainNavBar from "../../componets/common/navbar/Navbar";
import MainRecomm from "../../componets/content/MainRecomm/MainRecomm";
import MainActivity from "../../componets/content/MainActivity/MainActivity";
import MainGoodsList from "../../componets/content/MainGoodsList/MainGoodsList";
import MainInfo from "../../componets/content/MainInfo/MainInfo";
import Scroll from "../../componets/common/Scroll/Scroll";
import swiper2 from "../../componets/common/swiper/swiper2";
import Swiper from "../../componets/common/swiper/Swiper";
import { swiperreq, recommreq, getPopularList } from "../../network/homereq";
import debounce from "../../common/debounce";
import { backTop } from "../../common/mixin";
export default {
  mixins: [backTop],
  data() {
    return {
      char: "",
      saveY: 0,
      swiperItem: null,
      infoIsShow: false,
      imglist: [],
      recoImgList: [],
      sle: "popular",
      goods: {
        popular: { page: 1, list: [] },
        new: { page: 1, list: [] },
        featured: { page: 1, list: [] },
      },
    };
  },
  created() {
    // 尽量在生命周期函数中调用 methods 里的方法
    swiperreq().then((data) => {
      this.imglist = data;
    });
    recommreq().then((data) => {
      this.recoImgList = data;
    });
    this.getpopularList();
    this.getnewList();
    this.getfeaturedList();
  },
  mounted() {
    //去抖动的封装函数
    debounce(this.$bus, this.$refs.scroll.scroll);
  },
  methods: {
    search() {
      this.char = this.char.replace(/\s+/g, "");
      if (this.char != "") {
        this.$router.push({ path: "/search", query: { text: this.char } });
      } else {
        this.$router.push({ path: "/search", query: { text: "篮球鞋" } });
      }
    },
    shutInfo() {
      this.infoIsShow = false;
    },
    ctoItemDetails(item) {
      // 进入 swiper 详情页
      this.getswiperItem(item.name, (data) => {
        this.swiperItem = {
          data,
          item: item,
        };
        this.infoIsShow = true;
      });
    },
    pullingUp() {
      // console.log("ok");
      this["get" + this.sle + "List"]();
    },
    cswitch(sle) {
      this.sle = sle;
    },
    getpopularList() {
      getPopularList({
        url: "goods",
        params: {
          class: "篮球",
          page: this.goods.popular.page,
        },
      }).then((data) => {
        this.goods.popular.list = data;
        this.goods.popular.page += 1;
      });
    },
    getnewList() {
      getPopularList({
        url: "goods",
        params: {
          class: "新款",
          page: this.goods.popular.page,
        },
      }).then((data) => {
        this.goods.new.list = data;
        this.goods.new.page += 1;
      });
    },
    getfeaturedList() {
      getPopularList({
        url: "goods",
        params: {
          class: "精选",
          page: this.goods.popular.page,
        },
      }).then((data) => {
        this.goods.featured.list = data;
        this.goods.featured.page += 1;
      });
    },
    getswiperItem(name, callback) {
      getPopularList({
        url: "goods",
        params: {
          class: name,
          page: 1,
        },
      }).then((data) => {
        callback(data);
      });
    },
  },
  activated() {
    // console.log(this.saveY);
    //先刷新 scroll 再获取高度会解决切换后重新回到顶部的问题
    this.$refs.scroll && this.$refs.scroll.scroll.refresh();
    this.$refs.scroll && this.$refs.scroll.scroll.scrollTo(0, this.saveY, 0);
  },
  deactivated() {
    // console.log(this.$refs.scroll.scroll.startY);
    this.saveY = this.$refs.scroll.scroll.startY;
  },
  components: {
    MainSwitch,
    MainNavBar,
    MainRecomm,
    MainActivity,
    MainGoodsList,
    MainInfo,
    Swiper,
    Scroll,
    swiper2,
    // Activity,
  },
};
</script>
<style lang="less">
.swiper-wrapper {
  padding-top: 56px;
}
.body {
  position: relative;
  overflow: hidden;
  width: 100%;
  padding: 0 7px;
}
.nav-bar {
  z-index: 100;
  padding: 6px 7px;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
}
.left {
  i {
    font-size: 22px;
    position: absolute;
    left: 14px;
    color: rgb(255, 70, 70);
  }
  line-height: 44px;
  input {
    background: none;
    outline: none;
    border: none;
    height: 32px;
    width: 100%;
    border: 1px solid rgb(216, 215, 215);
    padding-left: 34px;
    letter-spacing: 1px;
    color: rgb(90, 90, 90);
    caret-color: rgb(255, 70, 70);
    font-size: 14px;
    -webkit-appearance: none;
  }
}
.right {
  div {
    padding-top: 3px;
    padding-left: 3px;
  }
  p {
    color: rgb(134, 134, 134);
    font-size: 10px;
    letter-spacing: 1px;
  }
  i {
    font-size: 20px;
    color: rgb(104, 104, 104);
  }
}
</style>