<template>
  <div class="viewer">
    <vc-viewer
      :infoBox="true"
      :shouldAnimate="true"
      :animation="true"
      :timeline="true"
      @ready="ready"
    >
      <vc-navigation></vc-navigation>
      <vc-layer-imagery>
        <vc-provider-imagery-bingmaps
          url="https://dev.virtualearth.net"
          bmKey="AgcbDCAOb9zMfquaT4Z-MdHX4AsHUNvs7xgdHefEA5myMHxZk87NTNgdLbG90IE-"
          mapStyle="Aerial"
        ></vc-provider-imagery-bingmaps>
      </vc-layer-imagery>
      <!-- <vc-provider-terrain-cesium ref="terrain" :requestWaterMask="requestWaterMask"></vc-provider-terrain-cesium> -->
      <!-- <vc-collection-primitive-point :points="points"></vc-collection-primitive-point> -->
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
      </vc-collection-primitive-point>-->

      <!-- <vc-primitive-polyline-ground :appearance="appearance">
        <vc-instance-geometry>
          <vc-geometry-polyline-ground ref="polylineGround" :positions="positions" :width="10"></vc-geometry-polyline-ground>
        </vc-instance-geometry>
      </vc-primitive-polyline-ground>-->

      <!-- <vc-datasource-kml ref="vcKml"  @ready="datasourceReady" data="./bikeRide.kml" :show="show"></vc-datasource-kml> -->

      

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
      <!-- <el-select v-model="selectValue" placeholder="请选择线路" @change="selectChange">
        <el-option
          v-for="item in selectOptions"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        ></el-option>
      </el-select>-->

      <el-button @click="showKml">{{showFlagKml ? '隐藏KML' : '显示KML' }}</el-button>
      <el-button @click="showFlight">{{showFlagFlight ? '隐藏Flight' : '显示Flight' }}</el-button>

      <!-- <el-button @click="showFlight">{{showFlagFlight ? '隐藏Flight' : '显示Flight' }}</el-button> -->

      <!-- <el-button @click="toggle('handlerPoint')">{{ pointDrawing ? '停止' : '点' }}</el-button>
      <el-button @click="toggle('handlerLine')">{{ polylineDrawing ? '停止' : '线' }}</el-button>
      <el-button @click="toggle('handlerPolygon')">{{ polygonDrawing ? '停止' : '面' }}</el-button>
      <el-button @click="jump">喜马拉雅</el-button>
      <el-button @click="clear">清除</el-button>-->
    </div>
  </div>
</template>
<script>
import url from "@/assets/img/railway.png";
export default {
  data() {
    return {
      showFlagKml: false,
      showFlagFlight: false,
      show: true,
      selectValue: "",
      selectOptions: [
        {
          value: "0",
          label: "成兰线1"
        },
        {
          value: "1",
          label: "成兰线2"
        },
        {
          value: "2",
          label: "成兰线3"
        },
        {
          value: "3",
          label: "成兰线4"
        },
        {
          value: "4",
          label: "成兰线5"
        }
      ],
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
      ],
      appearance: {},
      geometryInstances: {},
      positions: [
        { lng: 100.1340164450331, lat: 31.05494287836128 },
        { lng: 108.08821010582645, lat: 31.05494287836128 },
        { lng: 108.07821010582645, lat: 31.05494287836128 }
      ],
      dataSourcePromise: null
    };
  },
  mounted() {
    console.log(`this.viewer==`, this.viewer);
    Promise.all([this.$refs.vcKml.createPromise]).then((instances) => {
        instances[0].viewer.zoomTo(instances[0].viewer.entities)

        console.log(` instances[0].viewer.zoomTo(instances[0].viewer.entities)===`, instances[0].viewer.zoomTo(instances[0].viewer.entities))
      })
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

      // 显示线
      this.appearance = new Cesium.PolylineMaterialAppearance();

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

      const points = [];
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
        let point = {};
        point.position = {
          lng: Math.random() * 40 + 85,
          lat: Math.random() * 30 + 21
        };
        point.color = "rgb(255,229,0)";
        point.pixelSize = 8;
        points.push(point);
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
    movingEvt(windowPosition) {},
    drawEvt(result) {
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
    },

    // 显示线
    showLine() {
      this.$refs.polylineGround.createPromise.then(
        ({ Cesium, viewer, cesiumObject }) => {
          console.log(222, Cesium, viewer, cesiumObject);
          const boundingSphere = Cesium.BoundingSphere.fromPoints(
            cesiumObject._positions
          );
          viewer.scene.camera.flyToBoundingSphere(boundingSphere);
        }
      );
    },

    //显示飞行

    showFlight() {
      const { Cesium, viewer } = this.cesiumInstance;

      let options = {
        camera: viewer.scene.camera,
        canvas: viewer.scene.canvas
      };

    let xxoo=  viewer.dataSources
        .add(Cesium.KmlDataSource.load("./bikeRide.kml", options))
        .then(function(dataSource) {
          console.log(`dataSource==`,dataSource)
          viewer.clock.shouldAnimate = false;
          // var rider = dataSource.entities.getById("tour");
          let rider = new Cesium.Entity();
          viewer.flyTo(rider).then(function() {
            viewer.trackedEntity = rider;
            viewer.selectedEntity = viewer.trackedEntity;
            viewer.clock.multiplier = 30;
            viewer.clock.shouldAnimate = true;

            viewer.zoomTo(dataSource.viewer.entities)

          });
        });

      //   xxoo = viewer.then((instances) => {
      //     console.log(`instances==`,instances)
      //   instances[0].viewer.zoomTo(instances[0].viewer.entities)
      // })


      viewer.clock.clockRange = Cesium.ClockRange.UNBOUNDED;
      viewer.clock.clockStep = Cesium.ClockStep.SYSTEM_CLOCK;

      console.log(`2222222`, viewer.clock.clockRange, viewer.clock.clockStep);

      // this.showFlagFlight = !this.showFlagFlight;
    },

    //显示kml

    showKml() {
      const { Cesium, viewer } = this.cesiumInstance;

      console.log(
        `this.showFlagKml==`,
        this.showFlagKml,
        this.dataSourcePromise
      );
      if (this.showFlagKml) {
        viewer.dataSources.removeAll();
        this.dataSourcePromise = null;
        this.showFlagKml = !this.showFlagKml;
      } else {
        this.showFlagKml = !this.showFlagKml;

        let options = {
          camera: viewer.scene.camera,
          canvas: viewer.scene.canvas
        };

        this.dataSourcePromise = viewer.dataSources.add(
          Cesium.KmlDataSource.load("./proj.kml", options)
        );

        // 开启聚合

        this.dataSourcePromise.then(dataSource => {
          console.log(2222, dataSource);
          var pixelRange = 15;
          var minimumClusterSize = 3;
          var enabled = true;

          dataSource.clustering.enabled = enabled;
          dataSource.clustering.pixelRange = pixelRange;
          dataSource.clustering.minimumClusterSize = minimumClusterSize;

          var removeListener;

          var pinBuilder = new Cesium.PinBuilder();
          var pin50 = pinBuilder.fromUrl("./bigHose.png", Cesium.Color.RED, 48);
          var pin40 = pinBuilder.fromUrl(
            "./bigHose.png",
            Cesium.Color.ORANGE,
            48
          );
          var pin30 = pinBuilder.fromUrl(
            "./bigHose.png",
            Cesium.Color.YELLOW,
            48
          );
          var pin20 = pinBuilder.fromUrl(
            "./bigHose.png",
            Cesium.Color.GREEN,
            48
          );
          var pin10 = pinBuilder.fromUrl(
            "./bigHose.png",
            Cesium.Color.BLUE,
            48
          );

          var singleDigitPins = new Array(8);
          for (var i = 0; i < singleDigitPins.length; ++i) {
            singleDigitPins[i] = pinBuilder.fromUrl(
              "./bigHose.png",
              Cesium.Color.BLUE,
              48
            );
            // .fromText("" + (i + 2), Cesium.Color.VIOLET, 48)
            // .toDataURL();
          }

          function customStyle() {
            if (Cesium.defined(removeListener)) {
              removeListener();
              removeListener = undefined;
            } else {
              removeListener = dataSource.clustering.clusterEvent.addEventListener(
                function(clusteredEntities, cluster) {
                  cluster.label.show = false;
                  cluster.billboard.show = true;
                  cluster.billboard.id = cluster.label.id;
                  cluster.billboard.verticalOrigin =
                    Cesium.VerticalOrigin.BOTTOM;

                  if (clusteredEntities.length >= 50) {
                    cluster.billboard.image = pin50;
                  } else if (clusteredEntities.length >= 40) {
                    cluster.billboard.image = pin40;
                  } else if (clusteredEntities.length >= 30) {
                    cluster.billboard.image = pin30;
                  } else if (clusteredEntities.length >= 20) {
                    cluster.billboard.image = pin20;
                  } else if (clusteredEntities.length >= 10) {
                    cluster.billboard.image = pin10;
                  } else {
                    cluster.billboard.image =
                      singleDigitPins[clusteredEntities.length - 2];
                  }
                }
              );
            }

            // force a re-cluster with the new styling
            var pixelRange = dataSource.clustering.pixelRange;
            dataSource.clustering.pixelRange = 0;
            dataSource.clustering.pixelRange = pixelRange;
          }

          customStyle();
        });
      }
    },

    // select 选择线路
    selectChange(val) {
      val === 1 && this.showLine();
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