<template>
  <div class="map-warpper" :class="wrapper">
    <div id="container" style="width: 100%;height: 100%" />
    <transition @enter="enter" @after-enter="afterEnter" @leave="leave">
      <div v-if="show" class="animate-warpper" :style="{backgroundImage: `url(${coverImg})`}">
        <div class="animate-title">这是一个标题</div>
        <div>这是一个内容</div>
      </div>
    </transition>
    <transition @enter="coverEnter" @leave="coverLeave">
      <div v-if="coverShow" class="cover-warpper" @touchmove="touchmove">
        <div class="cover-title" :style="{backgroundImage: `url(${scrollImg})`}" />
      </div>
    </transition>

    <transition enter-active-class="animated fadeIn" leave-active-class="animated fadeIn">
      <div v-show="FirstTipShow" class="first-tip-warpper">
        <div class="first-tip-dot" :style="{backgroundImage: `url(${dotImg})`}"></div>
        <div class="first-tip" :style="{backgroundImage: `url(${bg1Img})`}"></div>
      </div>
    </transition>
    <transition
      enter-active-class="animated fadeOutRight"
      leave-active-class="animated fadeOutRight"
    >
      <div v-show="secondTipShow" class="second-tip-warpper">
        <div class="second-tip-dot" :style="{backgroundImage: `url(${dotImg})`}"></div>
        <div class="second-tip" :style="{backgroundImage: `url(${bg1Img})`}"></div>
      </div>
    </transition>
  </div>
</template>

<script>
import Img from "./scrollTop.png";
import coverImg from "./cover.png";
import dot from "./dot.png";
import bg1 from "./show_bg_1.png";

console.log(Img);
export default {
  name: "Map",
  data() {
    return {
      show: false,
      coverShow: false,
      FirstTipShow: false,
      secondTipShow: false,
      coverImg: coverImg,
      scrollImg: Img,
      dotImg: dot,
      bg1Img: bg1,
      isMobile: /iPad|iPod|iPhone|Android/.test(navigator.userAgent),
      AMap: window.AMap,
      zhouShanPos: [122.106863, 30.016028],
      wenzhouPos: [120.672111, 28.000575],
      hangZhouPos: [120.153576, 30.287459]
    };
  },
  computed: {
    wrapper: vm => (vm.isMobile ? "amap-ui-mobile" : "amap-ui-pc")
  },
  props: {
    msg: String
  },
  mounted() {
    this.init();
    //行政区划查询
    var opts = {
      subdistrict: 2, //返回下一级行政区
      showbiz: false //最后一级返回街道信息
    };
    let district = new this.AMap.DistrictSearch(opts); //注意：需要使用插件同步下发功能才能这样直接使用
    console.log(district);
    district.search("中国", function(status, result) {
      if (status == "complete") {
        console.log(result);
      }
    });
  },

  methods: {
    // set map
    init() {
      this.map = new this.AMap.Map("container", {
        center: this.hangZhouPos,
        gridMapForeign: true,
        resizeEnable: true,
        // zoomEnable: false,
        mapStyle: "amap://styles/blue",
        zoom: 7
      });
      // this.map.setFeatures(["road", "point", "bg"]);
      // zoom listening//basic UI control
      window.AMapUI.loadUI(["control/BasicControl"], BasicControl => {
        //添加一个缩放控件
        this.map.addControl(
          new BasicControl.Zoom({
            position: "rb",
            theme: "dark"
          })
        );
      });
      this.showFirstTip(this.map);
    },
    showFirstTip() {
      const base = 1000;
      const showTipTime = 2000;
      setTimeout(() => {
        this.map.setCenter(this.wenzhouPos);
        this.map.panBy(0, 200);
        setTimeout(() => {
          this.FirstTipShow = true;
        }, 500);
      }, base);
      setTimeout(() => {
        this.map.setCenter(this.zhouShanPos);
        this.FirstTipShow = false;
        setTimeout(() => {
          this.secondTipShow = true;
          const center = this.map.getCenter();
          var lnglat = new this.AMap.LngLat(center.lng, center.lat);
          var pixel = this.map.lngLatToContainer(lnglat);
          console.log(pixel);
          const y = pixel.y - 200;
          const firstTip = document.querySelector(".second-tip-warpper");
          firstTip.style.top = `${y}px`;
        }, 500);
      }, base + showTipTime);
      setTimeout(() => {
        this.map.setCenter(this.hangZhouPos);
        this.secondTipShow = false;
        this.show = true;
      }, base + showTipTime * 2);
    },
    // 标题出来的动画
    enter(el, done) {
      // el.offsetWidth 会强制刷新动画必须加，要不动画出不来
      el.offsetWidth;
      // 表示动画开始之后的样式 这里可以设置小球完成动画的结束状态
      el.style.opacity = 0.8;
      const children = Array.from(el.children);
      children.forEach((e, i) => {
        setTimeout(() => {
          console.log(e);
          e.style.opacity = 0.8;
          e.style.bottom = 0;
          if (i === children.length - 1) {
            // 这里的 done 就是 afterEnter 的引用
            done();
          }
        }, i + i * 1500);
      });
    },
    afterEnter() {
      // 动画完成之后
      setTimeout(() => {
        this.show = false;
      }, 1500);
    },
    leave(el, done) {
      window.Velocity(
        el,
        { transform: "scale(0)", opacity: 0 },
        { duration: 500 }
      );
      window.Velocity(el, { opacity: 0 }, { complete: done });
      setTimeout(() => {
        this.coverShow = true;
      }, 500);
    },
    // 向上滑动的悬浮成出来的动画
    coverEnter(el) {
      window.Velocity(el, { opacity: 1 }, { duration: 500 });
    },
    coverLeave(el) {
      window.Velocity(el, { opacity: 0 }, { duration: 500 });
      setTimeout(() => {
        console.log("调用接口");
      });
    },
    touchmove() {
      this.coverShow = false;
    }
  }
};

//fb_detect
</script>




<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.map-warpper {
  width: 100%;
  height: 100%;
  position: relative;
}
.animate-warpper {
  position: absolute;
  top: 40%;
  left: 0;
  width: 100%;
  height: 60%;
  opacity: 0;
  transition: all 1s ease;
  color: #ccc;
}
.animate-warpper div {
  opacity: 0;
  position: relative;
  bottom: -20px;
  transition: all 1s ease;
}
.animate-title {
  font-size: 30px;
}
.cover-warpper {
  position: absolute;
  left: 0%;
  bottom: 0%;
  width: 100%;
  height: 100%;
  z-index: 999999;
  background-color: rgba(0, 0, 0, 0.7);
  transition: all 1s ease;
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  opacity: 0;
}
.cover-title {
  position: absolute;
  left: 0%;
  bottom: 0%;
  height: 300px;
  width: 100%;
  background-repeat: no-repeat;
  background-position: center;
}
.first-tip-warpper {
  position: absolute;
  left: 50%;
  top: 50%;
  height: 230px;
  width: 230px;
  margin-left: -15px;
}
.first-tip-dot {
  position: absolute;
  left: 0;
  top: 200px;
  height: 30px;
  width: 30px;
  background-repeat: no-repeat;
  background-position: center;
}
.first-tip {
  height: 200px;
  width: 168.75px;
  background-repeat: no-repeat;
  background-position: center;
  background-size: 100% 100%;
}

.second-tip-warpper {
  position: absolute;
  left: 50%;
  top: 50%;
  height: 230px;
  width: 230px;
  margin-left: -15px;
}
.second-tip-dot {
  position: absolute;
  left: 0;
  top: 200px;
  height: 30px;
  width: 30px;
  background-repeat: no-repeat;
  background-position: center;
}
.second-tip {
  height: 200px;
  width: 168.75px;
  background-repeat: no-repeat;
  background-position: center;
  background-size: 100% 100%;
}
</style>
