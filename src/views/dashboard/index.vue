<template>
  <div class="viewer">
    <vc-viewer infoBox="true" @ready="ready">
      <vc-navigation></vc-navigation>
      <vc-layer-imagery>
        <vc-provider-imagery-bingmaps
          url="https://dev.virtualearth.net"
          bmKey="AgcbDCAOb9zMfquaT4Z-MdHX4AsHUNvs7xgdHefEA5myMHxZk87NTNgdLbG90IE-"
          mapStyle="Aerial"
        ></vc-provider-imagery-bingmaps>
      </vc-layer-imagery>
      <vc-provider-terrain-cesium ref="terrain" :requestWaterMask="requestWaterMask"></vc-provider-terrain-cesium>
      <vc-collection-primitive-point :points="points"></vc-collection-primitive-point>
      <!-- <vc-collection-primitive-point>
        <template v-for="(polyline, index) of polylines">
          <template v-for="(position, subIndex) of polyline.positions">
            <vc-primitive-point
              :position="position"
              :key="'point' + index + 'position' + subIndex"
              :color="colorPoint"
              :pixelSize="32"
            ></vc-primitive-point>
          </template>
        </template>
      </vc-collection-primitive-point> -->

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
import url from "@/assets/img/railway.png";
export default {
  data() {
    return {
      cesiumInstance: null,
      railwayUrl: url,
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
      },
      points: [],
      colorPoint: {},
      polylines: [
        {
          positions: [
            {
              lng: 119.24884033203125,
              lat: 30.313117980957031,
              height: 1183.3186645507812
            },
            {
              lng: 119.24906555725647,
              lat: 30.312892755731806,
              height: 1183.3186645507812
            }
          ]
        },
        {
          positions: [
            {
              lng: 119.24884033203125,
              lat: 30.313392639160156,
              height: 1183.804443359375
            },
            {
              lng: 119.24906555725632,
              lat: 30.31316741393502,
              height: 1183.6849884241819
            }
          ]
        },
        {
          positions: [
            {
              lng: 119.24884033203125,
              lat: 30.313655853271484,
              height: 1184.2783203125
            },
            {
              lng: 119.24906555725632,
              lat: 30.313430628046348,
              height: 1184.1093236654997
            }
          ]
        }
      ]
    };
  },
  mounted() {
    console.log(`this.viewer==`, this.viewer);
  },
  methods: {
    ready(cesiumInstance) {
      this.cesiumInstance = cesiumInstance;
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

      // viewer.entities.add({
      //   id: "兰成铁路,始建于1999年，全长150公里。。。",
      //   position: Cesium.Cartesian3.fromDegrees(104.06, 30.67, 1000),
      //   billboard: new Cesium.BillboardGraphics({
      //     image: this.railwayUrl,
      //     scale: 0.8
      //   }),
      //   label: new Cesium.LabelGraphics({
      //     text: "兰成铁路",
      //     font: "24px sans-serif",
      //     horizontalOrigin: 1,
      //     outlineColor: new Cesium.Color(0, 0, 0, 1),
      //     outlineWidth: 2,
      //     pixelOffset: new Cesium.Cartesian2(17, -5),
      //     style: Cesium.LabelStyle.FILL
      //   })
      // });

       const points = []
      // const points = [
      //   {
      //     position: { lng: 109.287331, lat: 21.261814 },
      //     color: "red",
      //     pixelSize: 8
      //   },
      //   {
      //     position: { lng: 106.44062, lat: 29.514887 },
      //     color: "green",
      //     pixelSize: 9
      //   },{
      //     position: { lng: 106.448328, lat: 29.524316 },
      //     color: "cyan",
      //     pixelSize: 10
      //   }
      // ];
      for (var i = 0; i < 1500; i++) {
        let point = {}
        point.position = { lng: Math.random() * 40 + 85, lat: Math.random() * 30 + 21 }
        point.color = 'rgb(255,229,0)'
        point.pixelSize = 8
        points.push(point)
      }
      this.points = points;
      // this.colorPoint = Cesium.Color.fromCssColorString("rgb(255,229,0)");
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
      console.log(22222222, windowPosition);
    },
    drawEvt(result) {
      console.log(33333333333);
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
  width: 100%;
  height: 700px;
  max-height: calc(100% - 200px);
  position: relative;
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