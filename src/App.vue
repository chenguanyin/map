<template>
  <div id="app">
    <Map msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
import Map from "./components/Map.vue";

export default {
  name: "app",
  components: {
    Map
  }
};
</script>

<style>
body,
html,
#app {
  margin: 0;
  height: 100%;
  width: 100%;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
}

.amap-ui-control-theme-dark .amap-ui-control-zoom a,
.amap-ui-control-theme-dark .amap-ui-control-zoom-num {
  background-color: #333 !important;
  border-bottom-color: #333 !important;
  color: #9b9b9b !important;
}

.amap-ui-control-theme-dark .amap-ui-control-zoom {
  border-color: #000 !important;
}

.control-zoom a,
.amap-ui-control-theme-dark .amap-ui-control-zoom-num {
  background-color: #333;
  border-bottom-color: #333 !important;
  color: #9b9b9b !important;
}

.amap-ui-control-zoom {
  border: 1px solid #ccc;
  background-clip: padding-box;
  border-radius: 0px !important;
  width: 32px;
  overflow: hidden;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.amap-overlays,
.amap-markers,
.amap-marker {
  position: absolute;
  transform: translate(0, 0) !important;
  left: 0;
  top: 0;
}

.amap-icon {
  position: relative;
  z-index: 9999999 !important;
  overflow: hidden;
}

.amap-ui-mobile .amap-ui-control-zoom {
  width: 20px !important;
}

.amap-ui-mobile .amap-ui-control-zoom a,
.amap-ui-mobile .amap-ui-control-zoom-num {
  width: 20px !important;
  line-height: 18px !important;
  height: 25px !important;
}

.amap-ui-control-position-br,
.amap-ui-control-position-rb {
  bottom: 150px !important;
  right: 460px !important;
  top: unset !important;
}
.amap-ui-mobile .amap-ui-control-position-br,
.amap-ui-mobile .amap-ui-control-position-rb {
  bottom: unset !important;
  right: 6% !important;
  top: 8% !important;
}
.amap-ui-control-zoom a {
  font-size: 18px !important;
  font-family: verdana;
}
@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.fadeIn {
  -webkit-animation-name: fadeIn;
  animation-name: fadeIn;
}
</style>
