<template>
  <div class="map-warpper" :class="wrapper" :style="{backgroundImage: `url(${bgImg})`}">
    <!-- 第一屏 -->
    <img :src="homeIcon" alt="图标" class="page-icon" />
    <span class="total-num" v-show="listShow">参与点亮人数：{{totalNum}}</span>
    <div id="fristMap" v-show="!listShow"></div>
    <div class="fristMap-overlays" v-show="!listShow"></div>
    <transition @enter="enter" @leave="leave">
      <div v-if="show" class="animate-warpper">
        <div class="home-icon" :style="{backgroundImage: `url(${fristIcon})`}"></div>
        <div class="home-title" :style="{backgroundImage: `url(${homeTitle})`}"></div>
      </div>
    </transition>
    <transition @enter="coverEnter" @leave="coverLeave">
      <div v-if="coverShow" class="cover-warpper" @touchmove="touchmove">
        <div class="cover-title" :style="{backgroundImage: `url(${arrowImg})`}" />
      </div>
    </transition>
    <transition
      name="custom-classes-transition"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOut"
    >
      <div v-show="FirstTipShow" class="first-tip-warpper">
        <div class="first-tip-dot" :style="{backgroundImage: `url(${dotImg})`}"></div>
        <div class="first-tip" :style="{backgroundImage: `url(${bg1Img})`}"></div>
      </div>
    </transition>
    <transition
      name="custom-transition"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOut"
    >
      <div v-show="secondTipShow" class="second-tip-warpper">
        <div class="second-tip-dot" :style="{backgroundImage: `url(${dotImg})`}"></div>
        <div class="second-tip" :style="{backgroundImage: `url(${bg1Img})`}"></div>
      </div>
    </transition>

    <!-- 第二屏的东西 -->
    <div
      id="container"
      style="width: 100%;height: 100%; pointer-events: none;"
      v-show="showSecondMap"
    />
    <transition enter-active-class="animated fadeIn" leave-active-class="animated fadeOut">
      <div v-if="listShow" class="scale-wrapper">
        <div v-for="(item,index) in sliceMapData" :key="item.id" class="scale-wrapper-item">
          <div v-if="index < 5">
            <span :style="{backgroundColor: `${colors[index]}`}" class="scale-bgcolor"></span>
            <span>{{item.area}}</span>
          </div>
        </div>
      </div>
    </transition>
    <transition @before-enter="listBeforeEnter" @enter="listEnter" @leave="listLeave">
      <div v-show="listShow" class="list-warpper">
        <div class="list-title">
          <span :class="{ active: activeKey === 0 }" @click="activeKey = 0">代表委员说</span>
          <span :class="{ active: activeKey === 1 }" @click="activeKey = 1">大家说</span>
        </div>
        <ul v-if="activeKey === 0">
          <li v-for="(data, index) in newData" :key="data.id">
            <div>
              <strong>{{data.city}}-{{data.usr}}</strong>
              <div>{{data.record}}</div>
            </div>
            <span
              class="like-icon"
              :style="{backgroundImage: `url(${data.isActive? likeActiveImg:likeImg})`}"
              @click="likeHandle(data, index)"
            >{{data.like_nums}}</span>
          </li>
        </ul>
        <ul v-if="activeKey === 1">
          <textarea rows="2" v-model="ideaStr"></textarea>
          <div>
            <a class="btn" :class="{disabled: !ideaStr}" @click="sendIdeaStrHandle">发表</a>
          </div>
          <li v-for="data in commentData" :key="data.id">
            <div>{{data.content}}</div>
          </li>
        </ul>
      </div>
    </transition>
    <transition
      name="custom-transition"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOut"
    >
      <div v-show="modalShow" class="modal-warpper">
        <div>发送成功，会在审核之后显示！！！</div>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from "axios";
import Img from "./scrollTop.png";
import coverImg from "./cover.png";
import dot from "./dot.png";
import bg1 from "./show_bg_1.png";
import homeIcon from "./home_icon.png";
import homeTitle from "./home_title.png";
import like from "./like.png";
import likeActive from "./like_clicked.png";
import bgImg from "./bg.png";
import fristIcon from "./first_icon.png";
import arrow from "./arrow.png";
export default {
  name: "Map",
  data() {
    return {
      show: false,
      coverShow: false,
      FirstTipShow: false,
      secondTipShow: false,
      listShow: false,
      modalShow: false,
      bgImg: bgImg,
      fristIcon: fristIcon,
      arrowImg: arrow,
      ideaStr: "",
      activeKey: 0,
      totalNum: 0,
      citylIist: [{ name: "杭州", number: 1 }],
      homeTitle: homeTitle,
      homeIcon: homeIcon,
      coverImg: coverImg,
      scrollImg: Img,
      dotImg: dot,
      bg1Img: bg1,
      likeImg: like,
      likeActiveImg: likeActive,
      showSecondMap: false,
      isMobile: /iPad|iPod|iPhone|Android/.test(navigator.userAgent),
      AMap: window.AMap,
      zhouShanPos: [122.106863, 30.016028],
      wenzhouPos: [120.672111, 28.000575],
      hangZhouPos: [120.153576, 30.287459],
      mapData: [],
      newData: [],
      commentData: [],
      mapColors: [],
      colors: ["#CD2626", "#DC143C", "#CD5555", "#EE6363", "#EE6A50", "#de4e4e"]
    };
  },
  computed: {
    wrapper: vm => (vm.isMobile ? "amap-ui-mobile" : "amap-ui-pc"),
    sliceMapData: vm => vm.mapData.slice(0, 5)
  },
  props: {
    msg: String
  },
  mounted() {
    this.getData();
    this.getTotal();
    this.init();
  },

  methods: {
    // 页面初始化
    init() {
      this.show = true;
      this.initMap("fristMap");
      this.map.panBy(-20, -95);
      this.initPro(1);
      // this.showFirstTip();
    },
    // 初始化地图
    initMap(id, config = {}) {
      this.map = new this.AMap.Map(id, {
        center: this.hangZhouPos,
        gridMapForeign: true,
        resizeEnable: true,
        // zoomEnable: false,
        dragEnable: false,
        scrollWheel: false,
        mapStyle: "amap://styles/normal",
        zoom: 7,
        ...config
      });
      this.map.setFeatures(["point", "bg"]);
      const overlays = document.querySelector(".fristMap-overlays");
      overlays.style.display = "inline-block";
    },

    // 获取对应数据
    getData() {
      axios.get("/getNews").then(res => {
        const { model, success } = res.data;
        if (success) {
          const likeInfo = JSON.parse(localStorage.getItem("likeInfo") || "[]");
          model.forEach(v => {
            if (likeInfo.includes(v.id)) {
              v.isActive = true;
            }
          });
          this.newData = model;
        }
      });
      axios.get("/getComment").then(res => {
        const { model, success } = res.data;
        if (success) {
          this.commentData = model;
        }
      });

      axios.get("/mapdata").then(res => {
        const { model, success } = res.data;
        if (success) {
          model.sort((pre, next) => +next.value - +pre.value);
          this.mapData = model;
        }
      });
    },

    // 地图颜色改变。。
    initPro(dep) {
      dep = typeof dep == "undefined" ? 0 : dep;
      this.disProvince && this.disProvince.setMap(null);
      console.log(this.AMap);
      this.disProvince = new this.AMap.DistrictLayer.Province({
        zIndex: 12,
        adcode: ["330000"],
        depth: dep,
        styles: {
          fill: properties => {
            // properties为可用于做样式映射的字段，包含
            // NAME_CHN:中文名称
            // adcode_pro
            // adcode_cit
            // adcode
            return this.getColorByName(properties.NAME_CHN);
          },
          "province-stroke": "white",
          "city-stroke": "white", // 中国地级市边界
          "county-stroke": "rgba(255,255,255,0.5)" // 中国区县边界
        }
      });

      this.disProvince.setMap(this.map);
      console.log(this.disProvince);
    },

    // 设置地图颜色
    getColorByName(name) {
      console.log(name);
      if (name === "浙江省") return "#ca3c32";
      if (!this.mapColors[name]) {
        const index = this.mapData.findIndex(v => name.indexOf(v.area) > -1);
        this.mapColors[name] =
          this.colors[index] || this.colors[this.colors.length - 1];
      }
      return this.mapColors[name];
    },

    // 没有用到，设置遮罩层
    setPolygons(name) {
      //行政区划查询
      var opts = {
        subdistrict: 0, //返回下一级行政区
        extensions: "all", // 返回行政区边界坐标等具体信息
        level: "city", // 查询范围是区
        showbiz: false //最后一级返回街道信息
      };
      let district = new this.AMap.DistrictSearch(opts); //注意：需要使用插件同步下发功能才能这样直接使用
      district.search(name, (status, result) => {
        if (status == "complete") {
          console.log(result);
          const { boundaries } = result.districtList[0];
          if (!this.polygons) this.polygons = [];
          console.log(result.districtList[0]);
          if (boundaries) {
            for (var i = 0, l = boundaries.length; i < l; i++) {
              // 生成行政区划polygon
              var polygon = new this.AMap.Polygon({
                map: this.map,
                strokeWeight: 1,
                path: boundaries[i],
                fillOpacity: 1,
                fillColor: "#f5c652",
                strokeColor: "#fff"
              });
              this.polygons.push(polygon);
            }
            setTimeout(() => {
              this.polygons.forEach(v => v.setMap(null));
            }, 500);
          }
        }
      });
    },

    // 设置marker点
    setMaker() {
      //行政区划查询
      var opts = {
        subdistrict: 1, //返回下一级行政区
        level: "city" // 查询范围是区¸
      };
      let district = new this.AMap.DistrictSearch(opts); //注意：需要使用插件同步下发功能才能这样直接使用
      const addMaker = center => {
        if (!this.markers) this.markers = [];
        var circleMarker = new this.AMap.CircleMarker({
          center: center,
          radius: 3, //3D视图下，CircleMarker半径不要超过64px
          strokeColor: "white",
          strokeWeight: 1,
          strokeOpacity: 0.5,
          fillColor: "rgb(245,200,82)",
          fillOpacity: 0.8,
          zIndex: 10,
          bubble: true,
          cursor: "pointer"
          // clickable: true,
          // click: e => {
          //   console.log(e);
          // }
        });
        circleMarker.setMap(this.map);
        this.markers.push(circleMarker);
      };
      console.log(district);
      this.mapData.forEach((v, i) => {
        // var center = [v.lng, v.lat];
        // addMaker(center);
        district.search(v.area, (status, result) => {
          if (status == "complete") {
            const data = result.districtList[0].districtList;
            console.log(data, i - data.length);
            data.forEach(city => {
              var cityCenter = [city.center.lng, city.center.lat];
              if (cityCenter) {
                addMaker(cityCenter);
              }
            });
          }
        });
      });
    },

    // 设置气泡框的弹出
    showFirstTip() {
      const base = 500;
      const showTipTime = 4000;
      const height = window.innerHeight;
      setTimeout(() => {
        this.map.panBy(0, 50);
        setTimeout(() => {
          this.FirstTipShow = true;
          // const center = this.map.getCenter();
          // var lnglat = new this.AMap.LngLat(center.lng, center.lat);
          // var pixel = this.map.lngLatToContainer(lnglat);
          // console.log(pixel);
          // const y = pixel.y - 200;
          const firstTip = document.querySelector(".first-tip-warpper");
          firstTip.style.top = `${height / 2 - 160}px`;
        }, 500);
      }, base);
      setTimeout(() => {
        this.map.setCenter(this.zhouShanPos);
        this.map.panBy(0, -1);
        this.FirstTipShow = false;
        setTimeout(() => {
          this.secondTipShow = true;
          const firstTip = document.querySelector(".second-tip-warpper");
          firstTip.style.top = `${height / 2 - 210}px`;
        }, 500);
      }, base + showTipTime);
      setTimeout(() => {
        this.map.setCenter(this.hangZhouPos);
        this.map.panBy(0, -100);
        this.secondTipShow = false;
        this.coverShow = true;
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
            // this.showFirstTip();
            setTimeout(() => {
              el.style.transform = "scale(1.4)";
              setTimeout(() => {
                el.style.transform = "scale(0.4)";
                setTimeout(() => {
                  el.style.transform = "scale(1)";
                  this.coverShow = true;
                });
              }, 500);
            }, 500);
            done();
          }
        }, i + i * 1500);
      });
    },
    leave(el) {
      el.style.opacity = 0;
    },
    // 向上滑动的悬浮成出来的动画
    coverEnter(el) {
      el.style.opacity = 1;
    },
    coverLeave(el) {
      el.style.opacity = 0;
      setTimeout(() => {
        this.showSecondMap = true;
        this.initMap("container");
      }, 500);
      setTimeout(() => {
        this.listShow = true;
        console.log("调用接口");
        this.initPro(1);
        this.map.panBy(-10, -266);
        this.setMaker();
      }, 600);
    },
    touchmove() {
      const map = document.getElementById("fristMap");
      const overlays = document.querySelector(".fristMap-overlays");
      const title = document.querySelector(".animate-warpper");
      map.style.transform = "scale(0)";
      overlays.style.display = "none";
      title.style.top = "-100%";
      this.coverShow = false;
    },
    listBeforeEnter(el) {
      el.style.bottom = "-366px";
    },
    listEnter(el) {
      el.style.bottom = "0px";
    },
    listLeave() {
      console.log("控制list不消失");
    },

    // 获取总数
    getTotal() {
      axios.get("/getAllUser").then(res => {
        if (res.data && res.data.success) {
          this.totalNum = res.data.total;
        }
      });
    },

    // 点赞操作
    likeHandle(data, i) {
      if (this.newData[i].isActive) return;
      const { id, city } = data;
      axios.post("/addValue", { id }).then(res => {
        if (res.data && res.data.success) {
          this.newData[i].isActive = true;
          this.newData[i].like_nums = +this.newData[i].like_nums + 1;
          this.$set(this.newData, i, this.newData[i]);
          const likeList = this.newData.filter(v => v.isActive).map(v => v.id);
          localStorage.setItem("likeInfo", JSON.stringify(likeList));
          this.setPolygons(city);
        }
      });
    },
    // 发送说的内容
    sendIdeaStrHandle() {
      if (!this.ideaStr) return;
      axios.post("/comment", { content: this.ideaStr }).then(res => {
        if (res.data && res.data.success) {
          this.ideaStr = "";
          this.modalShow = true;
          setTimeout(() => {
            this.modalShow = false;
          }, 2000);
        }
      });
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
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
}
#fristMap {
  position: absolute;
  top: 23%;
  bottom: 25%;
  width: 100%;
  transition: all 0.5s ease;
  pointer-events: none;
}
.fristMap-overlays {
  position: absolute;
  top: 23%;
  bottom: 25%;
  z-index: 11;
  display: none;
  width: 20%;
  height: 10%;
  background-color: #fcf9f2;
}
.page-icon {
  width: 110px;
  position: absolute;
  top: 0;
  left: 2%;
  z-index: 111;
}
.total-num {
  display: inline-block;
  position: absolute;
  top: 0;
  right: 2%;
  padding-top: 5px;
  z-index: 111;
  font-size: 13px;
  color: #000;
}
.animate-warpper {
  position: absolute;
  top: 6%;
  left: 0;
  width: 100%;
  height: 18%;
  opacity: 0;
  transition: all 0.5s ease;
  color: #ccc;
  text-align: center;
}
.home-icon {
  width: 100%;
  height: 40%;
  background-repeat: no-repeat;
  background-position: top left;
  background-size: 70% 100%;
  transition: all 1s ease;
}
.home-title {
  width: 100%;
  height: 60%;
  background-repeat: no-repeat;
  transition: all 1s ease;
  background-position: 80% 0%;
  background-size: 70%;
}
.animate-warpper div {
  opacity: 0;
  position: relative;
  bottom: -20px;
}
.cover-warpper {
  position: absolute;
  left: 0%;
  bottom: 8%;
  width: 100%;
  height: 50px;
  z-index: 999999;
  transition: all 1s ease;
  opacity: 0;
  text-align: center;
}
.cover-title {
  height: 50px;
  width: 50px;
  display: inline-block;
  background-repeat: no-repeat;
  background-position: center;
  background-size: 35px;
  animation: start 1.5s infinite ease;
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
.list-warpper {
  height: 50%;
  color: #000;
  position: absolute;
  bottom: -50%;
  z-index: 99999;
  left: 0%;
  width: 100%;
  border-top-right-radius: 6px;
  transition: all 1s ease;
  line-height: 30px;
}
.list-warpper ul {
  list-style: none;
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  background-color: #fff;
  margin: 0;
  padding: 10px 20px 38px;
  overflow: auto;
}
.list-warpper ul li {
  width: 100%;
  font-size: 14px;
  padding: 10px 0;
  overflow: hidden;
  word-break: break-all;
  border-bottom: 1px solid #9c9c9c6b;
}
.list-warpper ul li:last-child {
  border-bottom: none;
}
.list-title {
  display: inline-block;
  line-height: 28px;
  height: 28px;
  background-color: #fff;
  font-size: 0;
  box-shadow: 1px -4px 8px #3735417a;
}
.list-title span {
  display: inline-block;
  font-size: 14px;
  padding: 0 10px;
  background: rgba(255, 0, 0, 0.7);
  color: #fff;
}
.list-title span.active {
  background: #ffffff;
  color: #000;
}
.like-icon {
  float: right;
  height: 14px;
  padding-left: 18px;
  background-repeat: no-repeat;
  background-size: 14px 100%;
  line-height: 16px;
}
textarea {
  border: 1px solid rgb(204, 204, 204);
  width: 98%;
  padding: 5px;
  font-size: 15px;
}
.btn {
  background: rgba(255, 0, 0, 0.7);
  padding: 0 20px;
  border-radius: 4px;
  color: #fff;
  float: right;
}
.btn.disabled {
  background: #ccc;
}
.modal-warpper {
  position: absolute;
  top: 10%;
  left: 0;
  width: 100%;
  text-align: center;
  color: #000;
}
.modal-warpper div {
  display: inline-block;
  padding: 5px 10px;
  background: #fff;
  border-radius: 5px;
  border: 1px solid #ccc;
}
.scale-wrapper {
  position: absolute;
  top: 8%;
  left: 10px;
  color: #000;
  font-size: 12px;
  border: 1px solid #ccc;
}
.scale-wrapper-item {
  padding: 3px 5px;
  border-bottom: 1px solid #ccc;
  background: #fff;
}
.scale-wrapper-item:last-child {
  border-bottom: none;
}
.scale-bgcolor {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  margin-right: 10px;
}

@keyframes start {
  0%,
  30% {
    opacity: 1;
    transform: translateY(0px) rotate(-90deg);
  }
  100% {
    opacity: 1;
    transform: translateY(-45px) rotate(-90deg);
  }
}
</style>
