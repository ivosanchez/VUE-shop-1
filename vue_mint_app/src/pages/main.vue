<template>
  <div class="box">
    <!-- 下拉刷新上拉加载组件 -->
    <!-- 公共头部组件 -->
    <header-two :title="newTitle"></header-two>
    <!-- <mt-loadmore
      :top-method="loadTop"
      :bottom-method="loadBottom"
      :bottom-all-loaded="allLoaded"
      ref="loadmore"
      :distance-index="1"
    > -->
    <div style="height:100vh">

 <vue-better-scroll class="wrapper"
                         ref="scroll"
                         :scrollbar="scrollbarObj"
                         :pullDownRefresh="pullDownRefreshObj"
                         :pullUpLoad="pullUpLoadObj"
                         :startY="parseInt(startY)"
                         @pullingDown="onPullingDown"
                         @pullingUp="onPullingUp">



        <div class="main">
            <div class="lunbo">
            <mt-swipe :auto="4000">
                <mt-swipe-item v-for="(item,key,index) in swipers" :key="index">
                <img v-bind:src="item">
                </mt-swipe-item>
            </mt-swipe>
            </div>
            <div class="items">
            <div class="item" @click="xuangou('choose','选购')">
                <div class="item_img">
                <img src="../../static/images/SELECT.png">
                </div>
                <span class="item_title">选购</span>
            </div>
            <div class="item" @click="xuangou('need','真心想要')">
                <div class="item_img">
                <img src="../../static/images/lipin.png">
                </div>
                <span class="item_title">真心想要</span>
            </div>
            <div class="item" @click="xuangou('qiangou','限时抢购')">
                <div class="item_img">
                <img src="../../static/images/xianshi.png">
                </div>
                <span class="item_title">限时抢购</span>
            </div>
            <div class="item" @click="xuangou('fabu','新品发布')">
                <div class="item_img">
                <img src="../../static/images/news.png">
                </div>
                <span class="item_title">新品发布</span>
            </div>
            <div class="item" @click="xuangou('mifenka','米粉卡')">
                <div class="item_img">
                <img src="../../static/images/ka.png">
                </div>
                <span class="item_title">米粉卡</span>
            </div>
            </div>
            <div class="shop_mall">
            <div
                class="mall_item"
                @click="godetails(item.id)"
                v-for="(item,index) in items"
                :id="item.id"
                :key="index"
            >
                <img v-bind:src="item.img" v-lazy="item.img">
                <!-- 懒加载图片 -->
                <!-- <img v-lazy="item.img" /> -->
                <p class="mall_title">{{item.name}}</p>
                <div class="mall_item_all">
                <div class="mall_item_all_left">
                    ￥
                    <span class="price">{{item.Price}}</span>
                </div>
                <div class="mall_item_all_right">1人喜欢</div>
                </div>
            </div>
            </div>
        </div>
  </vue-better-scroll>
    </div>
  
    <!-- </mt-loadmore> -->
    <!-- 上滑 -->
    <div class="tops" v-if="showtop" @click="top" id="btn">
      <i class="iconfont icon-tubiao02 tops_size"></i>
    </div>
    <!-- 底部组件 -->
    <footer-bar class="footer"></footer-bar>
  </div>
</template>

<!-- <footer-bar class="footer"></footer-bar> -->
<script>
// 引入组件
import Vue from "vue";
import Footer from "../components/FooterBar";
import headerTwo from "../components/headerTwo.vue";
import global_ from "./Globaldata";
import { Indicator } from "mint-ui"; //引入mint ui
import VueBetterScroll  from 'vue2-better-scroll'  //上拉下拉组件
let count = 1
export default {
  // 组件开始
  components: {
    "vue-better-scroll": VueBetterScroll,
    "footer-bar": Footer,
    "header-two": headerTwo
  },
  // 组件结束
  data() {
    return {
         // 这个配置可以开启滚动条，默认为 false。当设置为 true 或者是一个 Object 的时候，都会开启滚动条，默认是会 fade 的
        scrollbarObj: {
          fade: true
        },
        // 这个配置用于做下拉刷新功能，默认为 false。当设置为 true 或者是一个 Object 的时候，可以开启下拉刷新，可以配置顶部下拉的距离（threshold） 来决定刷新时机以及回弹停留的距离（stop）
        pullDownRefreshObj: {
          threshold: 90,
          stop: 40
        },
        // 这个配置用于做上拉加载功能，默认为 false。当设置为 true 或者是一个 Object 的时候，可以开启上拉加载，可以配置离底部距离阈值（threshold）来决定开始加载的时机
        pullUpLoadObj: {
          threshold: 0,
          txt: {
            more: '加载更多',
            noMore: '没有更多数据了'
          }
        },
        startY: 0, // 纵轴方向初始化位置
        scrollToX: 0,
        scrollToY: 0,
        scrollToTime: 700,
        list: [],


      newTitle: "首页",
      allLoaded: false,
      scrollMode: "touch", //移动端弹性滚动效果，touch为弹性滚动，auto是非弹性滚动
      showtop: false, //top
      items: [],
      swipers: [
        "https://i8.mifile.cn/b2c-mimall-media/2fcbb6794e9f99d33bdf1f76cf380af0.jpg",
        "https://i8.mifile.cn/b2c-mimall-media/2be6be33c4131cfb2801ae41a2a84748.jpg",
        "https://i8.mifile.cn/b2c-mimall-media/d77913ecf914900557c0f9befedfc9bc.jpg"
      ],
      url:
        "https://www.easy-mock.com/mock/59e95287dd7e1a0a448c1102/example/todos",
      timer: null,
      //定义一个定时器
      isTop: true //定义一个布尔值，用于判断是否到达顶部
    };
  },
  watch: {},
  computed: {},
  created: function() {
    console.log("created", global_);
    this.getData();
  },
  beforeMount() {},
  mounted: function() {
    var _this = this;
    _this.onPullingDown()

    _this.$postHttp("https://www.easy-mock.com/mock/59e95287dd7e1a0a448c1102/example/demo", {} ).then(response => {
        console.log(response, "gggg");
      });

    var obtn = document.getElementById("btn"); //获取回到顶部按钮的ID
    var clientHeight = document.documentElement.clientHeight; //获取可视区域的高度
    window.onscroll = function() {
      //监听事件内容
      //获取滚动条的滚动高度
      var osTop = document.documentElement.scrollTop || document.body.scrollTop;
      if (osTop >= clientHeight) {
        //如果滚动高度大于可视区域高度，则显示回到顶部按钮
        _this.showtop = true;
      } else {
        //否则隐藏
        _this.showtop = false;
      } //主要用于判断当 点击回到顶部按钮后 滚动条在回滚过程中，若手动滚动滚动条，则清除定时器
      if (!_this.isTop) {
        clearInterval(_this.timer);
      }
      _this.isTop = false;
    };
  },
  beforeUpdate: function() {
    console.log("beforeUpdate");
  },
  updated() {
    console.log("updated");
  },
  beforeDestroy() {
    console.log("beforeDestroy");
  },
  destroyed() {
    console.log("destroyed");
  },
  methods: {
    async getData() {
      let response = await this.$getHttp(this.url, {});
      this.items = response;
    },
    // top
    loadTop() {
      console.log("刷新");
      var _this = this;
      // 加载更多数据
      setTimeout(function() {
        // 假设数据加载完
        _this.$refs.loadmore.onTopLoaded();
      }, 2000);
    },
    // bottom
    loadBottom() {
      var _this = this;
      console.log("bottom加载更多数据");
      // 加载更多数据
      setTimeout(function() {
        // 假设数据加载完
        _this.allLoaded = true; // 如果为true则不会在执行loadBottom（）方法,
        _this.$refs.loadmore.onBottomLoaded();
      }, 2000);
    },
    godetails: function(id) {
      this.$router.push({
        path: "details",
        query: {
          id: id
        }
      });
    },
    xuangou: function(path, name) {
      this.$router.push({
        path: path,
        query: {
          name: name
        }
      });
    },
    // need: function() {
    //     this.$router.push({
    //         path: 'need',
    //         query: {
    //             name: '真心想要'
    //         }
    //     })
    //     console.log(this)
    // },

    // 滑到顶部事件
    top: function() {
      let this_ = this;
      this_.timer = setInterval(function() {
        //获取滚动条的滚动高度
        var osTop =
          document.documentElement.scrollTop || document.body.scrollTop;
        //用于设置速度差，产生缓动的效果
        var speed = Math.floor(-osTop / 6);
        document.documentElement.scrollTop = document.body.scrollTop =
          osTop + speed;
        this_.isTop = true; //用于阻止滚动事件清除定时器
        if (osTop == 0) {
          clearInterval(this_.timer);
        }
      }, 30);

      // console.log("this", _this.$store.commit("checkoutData"))
      // _this.$store.commit("checkoutData");
    },
     // 模拟数据请求
      getDatas() {
        return new Promise(resolve => {
          setTimeout(() => {
            const arr = []
            for (let i = 0; i < 10; i++) {
              arr.push(count++)
            }
            resolve(arr)
          }, 1000)
        })
      },
      onPullingDown() {
        // 模拟下拉刷新
        console.log('下拉刷新')
        count = 0
        this.getDatas().then(res => {
          this.list = res
          this.$refs.scroll.forceUpdate(true)
        })
      },
      onPullingUp() {
        // 模拟上拉 加载更多数据
        console.log('上拉加载')
        this.getDatas().then(res => {
          this.list = this.list.concat(res)
          if (count < 30) {
            this.$refs.scroll.forceUpdate(true)
          } else {
            this.$refs.scroll.forceUpdate(false)
          }
        })
      }
  }
};
</script>

<style lang="css" scoped>
.box {
  padding-bottom: 0.8rem;
}

.tops {
  position: fixed;
  bottom: 1.2rem;
  right: 13px;
  z-index: 99;
  width: 0.88rem;
  height: 0.88rem;
}

.tops i {
  font-size: 0.65rem !important;
  color: #fe498f;
}

.lunbo {
  height: 5rem;
  margin-top: 0.2rem;
}

.mint-swipe-indicator {
  background: deeppink !important;
  opacity: 0.6 !important;
}

image[lazy="loading"] {
  width: 100%;
  height: 3.2rem;
  margin: auto;
}

.items {
  display: flex;
  justify-content: space-between;
  padding: 0.2rem 0.2rem;
  align-items: center;
  background: #fff;
}

.mint-swipe-item {
  width: 100%;
  height: 5rem;
}

.mint-swipe-item img {
  width: 100%;
  height: 100%;
}

.item_title {
  display: inline-block;
  margin-top: 0.2rem;
  font-size: 0.3rem;
  text-align: center;
}

.item_img img {
  width: 100%;
  height: 100%;
}

.item_img {
  width: 0.8rem;
  text-align: center;
}

.mall_item {
  width: 3.5rem;
  margin-top: 0.2rem;
  background: #fff;
}

.mall_item img {
  width: 100%;
  height: 3.2rem;
}

.shop_mall {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
}

.mall_title {
  text-align: center;
  font-size: 0.3rem;
  margin-top: 0.2rem;
}

.mall_item_all {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 0.2rem;
  padding: 0.2rem 0.1rem;
}

.mall_item_all_left {
  font-size: 0.4rem;
  color: red;
}

.price {
  font-size: 0.4rem;
}

.item {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.mall_item_all_right {
  font-size: 0.4rem;
}
</style>