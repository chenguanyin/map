<template>
  <div class="map-warpper" :class="wrapper">
    <img :src="homeIcon" alt="图标" class="page-icon" />
    <div id="container" style="width: 100%;height: 100%" />
    <transition @enter="enter" @after-enter="afterEnter" @leave="leave">
      <div v-if="show" class="animate-warpper" :style="{backgroundImage: `url(${coverImg})`}">
        <div class="home-title" :style="{backgroundImage: `url(${homeIcon})`}">
          <img :src="homeTitle" alt="标题" />
        </div>
        <div>文字。。。。</div>
      </div>
    </transition>
    <transition @enter="coverEnter" @leave="coverLeave">
      <div v-if="coverShow" class="cover-warpper" @touchmove="touchmove">
        <div class="cover-title" :style="{backgroundImage: `url(${scrollImg})`}" />
      </div>
    </transition>

    <transition
      name="custom-classes-transition"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOut"
    >
      <div v-if="FirstTipShow" class="first-tip-warpper">
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
    <transition @before-enter="listBeforeEnter" @enter="listEnter" @leave="listLeave">
      <div v-show="listShow" class="list-warpper">
        <div class="list-title">
          <span :class="{ active: activeKey === 0 }" @click="activeKey = 0">委员说</span>
          <span :class="{ active: activeKey === 1 }" @click="activeKey = 1">群众说</span>
        </div>
        <ul v-if="activeKey === 0">
          <li v-for="data in newData" :key="data.id">
            <div>{{data.record}}</div>
            <span
              class="like-icon"
              :style="{backgroundImage: `url(${data.isActive? likeActiveImg:likeImg})`}"
              @click="likeHandle(data.id)"
            >{{data.like_nums}}</span>
          </li>
        </ul>
        <ul v-if="activeKey === 1">
          <textarea cols="30" rows="3" v-model="ideaStr"></textarea>
          <div>
            <a class="btn" :class="{disabled: !ideaStr}" @click="sendIdeaStrHandle">发表</a>
          </div>
          <li v-for="data in newData" :key="data.id">
            <div>{{data.record}}</div>
          </li>
        </ul>
      </div>
    </transition>
    <transition
      name="custom-transition"
      enter-active-class="animated fadeIn"
      leave-active-class="animated fadeOut"
    >
      <div v-show="modalShow" class="modal-warpper">发送成功，会在审核之后显示！！！</div>
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
console.log(Img);
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
      ideaStr: "",
      activeKey: 0,
      citylIist: [{ name: "杭州", number: 1 }],
      homeTitle: homeTitle,
      homeIcon: homeIcon,
      coverImg: coverImg,
      scrollImg: Img,
      dotImg: dot,
      bg1Img: bg1,
      likeImg: like,
      likeActiveImg: likeActive,
      isMobile: /iPad|iPod|iPhone|Android/.test(navigator.userAgent),
      AMap: window.AMap,
      zhouShanPos: [122.106863, 30.016028],
      wenzhouPos: [120.672111, 28.000575],
      hangZhouPos: [120.153576, 30.287459],
      mapData: [
        { id: "00000001", area: "宁波", value: "1232" },
        { id: "00000002", area: "杭州", value: "12366" },
        { id: "00000003", area: "嘉兴", value: "12312" }
      ],
      newData: [
        {
          id: "00000001",
          record:
            "丁元竹：新常态下，在心态上必须适应更慢更持续的发展态势和更低的经济增长预期。",
          like_nums: "1223"
        },
        {
          id: "00000002",
          record: "贾晋京：简政放权确实有阻力，总理的“弹药”就是壮士断腕。",
          like_nums: "12312"
        },
        {
          id: "00000003",
          record:
            "李长安：“去产能”工作并非一蹴而就，“保就业”的任务也任重道远。",
          like_nums: "2423"
        }
      ],
      colors: []
    };
  },
  computed: {
    wrapper: vm => (vm.isMobile ? "amap-ui-mobile" : "amap-ui-pc")
  },
  props: {
    msg: String
  },
  mounted() {
    this.getData();
    this.init();
  },

  methods: {
    // set map
    init() {
      this.map = new this.AMap.Map("container", {
        center: this.hangZhouPos,
        gridMapForeign: true,
        resizeEnable: true,
        // zoomEnable: false,
        mapStyle: "amap://styles/normal",
        zoom: 7
      });
      this.map.setFeatures(["road", "point", "bg"]);
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
      this.initPro(1);
    },

    getData() {
      axios.get("/getNews", res => {
        console.log(res);
      });
      axios.get("/getComment", res => {
        console.log(res);
      });
    },

    getMapData() {
      axios.get("/mapdata", res => {
        console.log(res);
      });
    },

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
          "province-stroke": "cornflowerblue",
          "city-stroke": "white", // 中国地级市边界
          "county-stroke": "rgba(255,255,255,0.5)" // 中国区县边界
        }
      });

      this.disProvince.setMap(this.map);
      this.showFirstTip(this.map);
      // this.listShow = true;
    },

    getColorByName(name) {
      console.log(name);
      if (!this.colors[name]) {
        var gb = Math.random() * 155;
        this.colors[name] = `rgb(200, ${gb},${gb})`;
      }

      return this.colors[name];
    },
    setPolygons() {
      //行政区划查询
      var opts = {
        subdistrict: 2, //返回下一级行政区
        extensions: "all", // 返回行政区边界坐标等具体信息
        level: "province", // 查询范围是区
        showbiz: false //最后一级返回街道信息
      };
      let district = new this.AMap.DistrictSearch(opts); //注意：需要使用插件同步下发功能才能这样直接使用
      district.search("浙江省", (status, result) => {
        if (status == "complete") {
          console.log(result);
          const { boundaries, districtList } = result.districtList[0];
          var polygons = [];
          const obj = {};
          districtList.forEach(v => {
            obj[v.name] = {
              lng: v.center.lng,
              lat: v.center.lat
            };
          });
          console.log(JSON.stringify(obj));
          if (boundaries) {
            for (var i = 0, l = boundaries.length; i < l; i++) {
              //生成行政区划polygon
              var polygon = new this.AMap.Polygon({
                map: this.map,
                strokeWeight: 1,
                path: boundaries[i],
                fillOpacity: 0.7,
                fillColor: "rgba(255, 0, 0, 0.5)",
                strokeColor: "#fff"
              });
              polygons.push(polygon);
            }
          }
          // this.showFirstTip(this.map);
          this.listShow = true;
        }
      });
    },

    setMaker() {
      //行政区划查询
      var opts = {
        subdistrict: 2, //返回下一级行政区
        extensions: "all", // 返回行政区边界坐标等具体信息
        level: "district", // 查询范围是区
        showbiz: false //最后一级返回街道信息
      };
      let district = new this.AMap.DistrictSearch(opts); //注意：需要使用插件同步下发功能才能这样直接使用
      console.log(district);
      // district.search("浙江省", (status, result) => {
      //   if (status == "complete") {
      //     console.log(result);
      //     const { boundaries } = result.districtList[0];
      //     var outer = [
      //       new this.AMap.LngLat(-360, 90, true),
      //       new this.AMap.LngLat(-360, -90, true),
      //       new this.AMap.LngLat(360, -90, true),
      //       new this.AMap.LngLat(360, 90, true)
      //     ];
      //     var pathArray = [outer];
      //     pathArray.push.apply(pathArray, boundaries);
      //     var polygon = new this.AMap.Polygon({
      //       // pathL:pathArray,
      //       //线条颜色，使用16进制颜色代码赋值。默认值为#006600
      //       strokeColor: "rgb(20,164,173)",
      //       strokeWeight: 1,
      //       //轮廓线透明度，取值范围[0,1]，0表示完全透明，1表示不透明。默认为0.9
      //       strokeOpacity: 0.5,
      //       //多边形填充颜色，使用16进制颜色代码赋值，如：#FFAA00
      //       fillColor: "rgba(0,0,0)",
      //       //多边形填充透明度，取值范围[0,1]，0表示完全透明，1表示不透明。默认为0.9
      //       fillOpacity: 1,
      //       //轮廓线样式，实线:solid，虚线:dashed
      //       strokeStyle: "dashed",
      //       /*勾勒形状轮廓的虚线和间隙的样式，此属性在strokeStyle 为dashed 时有效， 此属性在
      //         ie9+浏览器有效 取值：
      //         实线：[0,0,0]
      //         虚线：[10,10] ，[10,10] 表示10个像素的实线和10个像素的空白（如此反复）组成的虚线
      //         点画线：[10,2,10]， [10,2,10] 表示10个像素的实线和2个像素的空白 + 10个像素的实
      //         线和10个像素的空白 （如此反复）组成的虚线*/
      //       strokeDasharray: [0, 0, 0]
      //     });
      //     polygon.setPath(pathArray);
      //     this.map.add(polygon);
      //     // const data = result.districtList[0].districtList.find(
      //     //   v => v.adcode == "330000"
      //     // );
      //     // data.districtList.forEach(v => {
      //     //   var center = [v.center.lng, v.center.lat];
      //     //   var circleMarker = new this.AMap.CircleMarker({
      //     //     center: center,
      //     //     radius: 15, //3D视图下，CircleMarker半径不要超过64px
      //     //     strokeColor: "white",
      //     //     strokeWeight: 2,
      //     //     strokeOpacity: 0.5,
      //     //     fillColor: "rgba(170,114,57,1)",
      //     //     fillOpacity: 0.8,
      //     //     zIndex: 10,
      //     //     bubble: true,
      //     //     cursor: "pointer",
      //     //     clickable: true,
      //     //     click: e => {
      //     //       console.log(e);
      //     //     }
      //     //   });
      //     //   circleMarker.setMap(this.map);
      //     //   circleMarker.on("click", function(e) {
      //     //     console.log(e);
      //     //   });
      //     // });
      //   }
      // });
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
        this.listShow = true;
        this.map.panBy(0, -266);
      });
    },
    touchmove() {
      this.coverShow = false;
    },
    listBeforeEnter(el) {
      el.style.bottom = "-366px";
    },
    listEnter(el) {
      window.Velocity(el, { bottom: "0px" }, { duration: 500 });
    },
    listLeave() {
      console.log("控制list不消失");
    },
    likeHandle(id) {
      console.log(id);
      axios.post("/addValue", { content: this.ideaStr }).then(res => {
        if (res.success) {
          alert("点赞成功");
        }
      });
    },
    sendIdeaStrHandle() {
      if (!this.ideaStr) return;
      axios.post("/comment", { content: this.ideaStr }).then(res => {
        if (res.success) {
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
}
.page-icon {
  width: 90px;
  position: absolute;
  top: 0;
  left: 2%;
  z-index: 111;
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
  text-align: center;
}
.home-title {
  width: 100%;
  height: 80px;
  padding-top: 10px;
  margin-bottom: 30px;
  background-repeat: no-repeat;
  background-position: 95% 0%;
  background-size: 80px;
}
.home-title img {
  width: 90%;
  height: 100%;
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
.list-warpper {
  height: 366px;
  color: #000;
  position: absolute;
  bottom: -336px;
  z-index: 99999;
  margin-left: 5%;
  margin-right: 5%;
  left: 0%;
  right: 0;
  border-top-right-radius: 6px;
  line-height: 30px;
}
.list-warpper ul {
  list-style: none;
  height: 338px;
  background-color: #fff;
  margin: 0;
  padding: 20px;
}
.list-warpper ul li {
  width: 100%;
  font-size: 14px;
  padding: 10px 0;
  overflow: hidden;
  word-break: break-all;
  border-bottom: 1px solid #9c9c9c6b;
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
}
.list-title span.active {
  background: rgba(255, 0, 0, 0.7);
  color: #fff;
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
}
</style>
