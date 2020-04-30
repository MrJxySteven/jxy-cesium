<template>
  <div class="viewer">
    <vc-viewer  infoBox="true" @ready="ready">
      <vc-navigation></vc-navigation>
      <vc-layer-imagery>
        <vc-provider-imagery-bingmaps
          url="https://dev.virtualearth.net"
          bmKey="AgcbDCAOb9zMfquaT4Z-MdHX4AsHUNvs7xgdHefEA5myMHxZk87NTNgdLbG90IE-"
          mapStyle="Aerial"
        ></vc-provider-imagery-bingmaps>
      </vc-layer-imagery>
      <vc-provider-terrain-cesium ref="terrain" :requestWaterMask="requestWaterMask"></vc-provider-terrain-cesium>

      <vc-handler-draw-point
        ref="handlerPoint"
        @activeEvt="activeEvt"
        @movingEvt="movingEvt"
        @drawEvt="drawEvt"
        pointColor="red"
      ></vc-handler-draw-point>
      <vc-handler-draw-polyline
        ref="handlerLine"
        :polylineMaterial="polylineMaterial"
        @activeEvt="activeEvt"
        @movingEvt="movingEvt"
        @drawEvt="drawEvt"
      ></vc-handler-draw-polyline>
      <vc-handler-draw-polygon
        ref="handlerPolygon"
        @activeEvt="activeEvt"
        @movingEvt="movingEvt"
        @drawEvt="drawEvt"
      ></vc-handler-draw-polygon>
    </vc-viewer>
    <div class="demo-tool">
      <el-button @click="toggle('handlerPoint')">{{ pointDrawing ? '停止' : '点' }}</el-button>
      <el-button @click="toggle('handlerLine')">{{ polylineDrawing ? '停止' : '线' }}</el-button>
      <el-button @click="toggle('handlerPolygon')">{{ polygonDrawing ? '停止' : '面' }}</el-button>
      <el-button @click="jump">喜马拉雅</el-button>
      <el-button @click="clear">清除</el-button>
    </div>
  </div>
</template>
<script>
import url from '@/assets/img/railway.png'
export default {
  data() {
    return {
      cesiumInstance:null,
      railwayUrl:url,
      pointDrawing: false,
      polylineDrawing: false,
      polygonDrawing: false,
      polylineMaterial: {
        fabric: {
          type: "PolylineDash",
          uniforms: {
            color: "blue"
          }
        }
      }
    };
  },
  mounted() {
    console.log(`this.viewer==`,this.viewer)
  },
  methods: {
    ready(cesiumInstance) {
      this.cesiumInstance = cesiumInstance
      const { Cesium, viewer } = cesiumInstance;
      // var target = new Cesium.Cartesian3(
      //   300770.50872389384,
      //   5634912.131394585,
      //   2978152.2865545116
      // );
      // var offset = new Cesium.Cartesian3(
      //   6344.974098678562,
      //   -793.3419798081741,
      //   2499.9508860763162
      // );
      // viewer.camera.lookAt(target, offset);
      viewer.camera.lookAtTransform(Cesium.Matrix4.IDENTITY);


      viewer.entities.add({
          id: '兰成铁路,始建于1999年，全长150公里。。。',
          position: Cesium.Cartesian3.fromDegrees(104.06, 30.67, 1000),
          billboard: new Cesium.BillboardGraphics({
            image: this.railwayUrl,
            scale: .8
          }),
          label: new Cesium.LabelGraphics({
            text: '兰成铁路',
            font: '24px sans-serif',
            horizontalOrigin: 1,
            outlineColor: new Cesium.Color(0, 0, 0, 1),
            outlineWidth: 2,
            pixelOffset: new Cesium.Cartesian2(17, -5),
            style: Cesium.LabelStyle.FILL
          })
        })

      
    },
    toggle(type) {
      console.log(type);
      this.$refs[type].drawing = !this.$refs[type].drawing;
    },
    clear() {
      this.$refs.handlerPoint.clear();
      this.$refs.handlerLine.clear();
      this.$refs.handlerPolygon.clear();
    },
    activeEvt(_) {
      this[_.type] = _.isActive;
    },
    movingEvt(windowPosition) {
      console.log(22222222,windowPosition)
    },
    drawEvt(result) {
      console.log(33333333333)
      // result.finished && false;
      console.log(result);
    },
    readyPromise(tileset) {
      const { viewer } = this.cesiumInstance;
      viewer.zoomTo(
        tileset,
        new Cesium.HeadingPitchRange(
          0.0,
          -0.5,
          tileset.boundingSphere.radius * 2.0
        )
      );
    },
    jump() {
       const { viewer } = this.cesiumInstance;
       var target = new Cesium.Cartesian3(
        300770.50872389384,
        5634912.131394585,
        2978152.2865545116
      );
      var offset = new Cesium.Cartesian3(
        6344.974098678562,
        -793.3419798081741,
        2499.9508860763162
      );
     viewer.camera.lookAt(target, offset);
    }

  }
};
</script>

<style lang="scss" scoped>
.viewer {
  width:100%;
  height:700px;
  max-height: calc(100% - 200px);
  position:relative;
}
.demo-tool {
  position: absolute;
  left: 1%;
  top: 1%;
  min-width: 185px;
  z-index: 100;
  color: #fff;
  background-color: #ccc;
}
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
.twipsy {
  display: block;
  position: absolute;
  visibility: visible;
  max-width: 200px;
  min-width: 100px;
  padding: 5px;
  font-size: 11px;
  z-index: 1000;
  opacity: 0.8;
  -khtml-opacity: 0.8;
  -moz-opacity: 0.8;
  filter: alpha(opacity=80);
}
.twipsy.left .twipsy-arrow {
  top: 50%;
  right: 0;
  margin-top: -5px;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 5px solid #000000;
}
.twipsy.right .twipsy-arrow {
  top: 50%;
  left: 0;
  margin-top: -5px;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-right: 5px solid #000000;
}
.twipsy-inner {
  padding: 3px 8px;
  background-color: #000000;
  color: white;
  text-align: center;
  max-width: 200px;
  text-decoration: none;
  -webkit-border-radius: 4px;
  -moz-border-radius: 4px;
  border-radius: 4px;
}
.twipsy-arrow {
  position: absolute;
  width: 0;
  height: 0;
}
</style>