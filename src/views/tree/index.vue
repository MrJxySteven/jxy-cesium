<template>
  <div class="viewer">
    <vc-viewer
      :shouldAnimate="true"
      :animation="true"
      :timeline="true"
      @ready="ready"
      :terrainProvider="terrainProvider"
    >
      <vc-entity
        ref="entity"
        :availability="availability"
        :position="position1"
        :orientation="orientation"
        :description="description"
      >
        <!-- <vc-graphics-path ref="path" :resolution="1" :material="material1" :width="10"></vc-graphics-path> -->

        <vc-datasource-kml ref="path"   @ready="readyKml"  data="./bikeRide.kml" :show="show"></vc-datasource-kml>

        <vc-graphics-model ref="model" :uri="uri1" :minimumPixelSize="64"></vc-graphics-model>
      </vc-entity>
      <vc-entity
        :key="'entity' + index"
        :position="position"
        v-for="(position, index) of positions"
      >
        <vc-graphics-point :pixelSize="8" color="BLUE" outlineColor="RED" :outlineWidth="5"></vc-graphics-point>
      </vc-entity>
      <vc-layer-imagery>
        <vc-provider-imagery-bingmaps
          url="https://dev.virtualearth.net"
          bmKey="AgcbDCAOb9zMfquaT4Z-MdHX4AsHUNvs7xgdHefEA5myMHxZk87NTNgdLbG90IE-"
          mapStyle="Aerial"
        ></vc-provider-imagery-bingmaps>
      </vc-layer-imagery>
    </vc-viewer>
    <div class="demo-tool">
      <el-button @click="viewTopDown">俯视图</el-button>
      <el-button @click="viewSide">自由视图</el-button>
      <el-button @click="viewAircraft">正视图</el-button>
      <el-button @click="kmlFlight">kmlFlight</el-button>
    </div>
  </div>
</template>
<script>
// import url from '../../assets/img/railway.png'
// import url from './Cesium_Air.gltf'
export default {
  data() {
    return {
      show: false,
      cesiumInstance: {},
      description: "兰成路线",
      model1: {},
      path1: {},
      material1: {},
      availability: {},
      position1: {},
      terrainProvider: {},
      orientation: {},
      uri1: "./Cesium_Air.gltf",
      // uri1:url,
      start: {},
      stop: {},
      positions: [],
      kmlObj: null
    };
  },
  mounted() {
    Promise.all([
      this.$refs.path.createPromise,
      this.$refs.model.createPromise
    ]).then(instances => {
      // instances[0].viewer.zoomTo(instances[0].viewer.entities)

      console.log(
        ` instances[0].viewer.zoomTo(instances[0].viewer.entities)===`,
        instances[0].viewer.zoomTo(instances[0].viewer.entities)
      );
    });
  },
  methods: {
    readyKml(cesiumInstance) {
      const { Cesium, viewer, cesiumObject } = cesiumInstance;

      this.kmlObj = cesiumObject

      // console.log(`this.kmlObj--`,this.kmlObj,  this.$refs.path.createPromise)
      // this.position1 = cesiumInstance
      // this.position1 = this.computeCirclularFlight(86.3553137781065, 41.2142425379045, 0.03)
    },
    ready(cesiumInstance) {
      const { Cesium, viewer } = cesiumInstance;
      this.cesiumInstance = cesiumInstance;
      this.terrainProvider = Cesium.createWorldTerrain();
      this.position1 = Cesium.Cartesian3.fromDegrees(114.0, 40.0, 1.0);
      //Enable lighting based on sun/moon positions
      viewer.scene.globe.enableLighting = true;
      //Enable depth testing so things behind the terrain disappear.
      viewer.scene.globe.depthTestAgainstTerrain = true;
      //Set the random number seed for consistent results.
      Cesium.Math.setRandomNumberSeed(3);
      //Set bounds of our simulation time
      this.start = Cesium.JulianDate.fromDate(new Date(2020, 4, 8, 4));
      this.stop = Cesium.JulianDate.addSeconds(
        this.start,
        180,
        new Cesium.JulianDate()
      );
      viewer.clock.startTime = this.start.clone();
      viewer.clock.stopTime = this.stop.clone();
      viewer.clock.currentTime = this.start.clone();
      viewer.clock.clockRange = Cesium.ClockRange.LOOP_STOP; //Loop at the end
      viewer.clock.multiplier = 10;
      viewer.timeline.zoomTo(this.start, this.stop);
      // this.position1 = this.computeCirclularFlight(
      //   86.3553137781065,
      //   41.2142425379045,
      //   0.03
      // );
      console.log(`this.position1==`, this.position1);

      //         let arrPos =[
      //           {lng: 86.3553137781065, lat: 40.2142425379045, height: 0 },
      //           {lng: 87.3553137781065, lat: 41.2142425379045, height: 0 },
      //           {lng: 88.3553137781065, lat: 42.2142425379045, height: 0 },
      //         ]

      //         this.position1 = arrPos
      // ss
      //         this.positions = arrPos
      this.availability = new Cesium.TimeIntervalCollection([
        new Cesium.TimeInterval({
          start: this.start,
          stop: this.stop
        })
      ]);
      this.orientation = new Cesium.VelocityOrientationProperty(this.position1);
      this.material1 = new Cesium.PolylineGlowMaterialProperty({
        glowPower: 0.1,
        color: Cesium.Color.RED
      });
    },

    kmlFlight() {
      this.show = true
      console.log(`this.kmlObj----`,this.kmlObj)
      this.position1 = this.computeCirclularFlight(
        86.3553137781065,
        41.2142425379045,
        0.03
      );
    },

    computeCirclularFlight(lon, lat, radius) {
        const { Cesium, viewer } = this.cesiumInstance
        let property = new Cesium.SampledPositionProperty()

        console.log(`this.kmlObj===`,this.kmlObj)

        this.positions.push(this.kmlObj)

        property.addSample(this.kmlObj)

        return property

        // for (let i = 0; i <= 360; i += 45) {
        //   let radians = Cesium.Math.toRadians(i)
        //   let time = Cesium.JulianDate.addSeconds(this.start, i, new Cesium.JulianDate())
        //   let position = Cesium.Cartesian3.fromDegrees(
        //     lon + radius * 1.5 * Math.cos(radians),
        //     lat + radius * Math.sin(radians),
        //     Cesium.Math.nextRandomNumber() * 500 + 1750
        //   )
        //   property.addSample(time, position)
        //   this.positions.push(position)
        // }
        return property
      },
    // computeCirclularFlight(lon, lat, radius) {
    //   let arrPos = [
    //     { lng: 86.3553137781065, lat: 40.2142425379045, height: 0 },
    //     { lng: 88.3553137781065, lat: 44.2142425379045, height: 0 },
    //     { lng: 90.3553137781065, lat: 46.2142425379045, height: 0 }
    //   ];

    //   const { Cesium, viewer } = this.cesiumInstance;
    //   let property = new Cesium.SampledPositionProperty();

    //   for(let i=0; i< arrPos.length; i++) {
    //     // let radians = Cesium.Math.toRadians(i)
    //       let time = Cesium.JulianDate.addSeconds(this.start, 90*i, new Cesium.JulianDate())
    //       let position = Cesium.Cartesian3.fromDegrees(
    //         arrPos[i].lng,
    //         arrPos[i].lat,
    //          Cesium.Math.nextRandomNumber() * 500 + 1750
    //       )

    //       console.log(`position==`,position)
    //       property.addSample(time, position)
    //       this.positions.push(position)
    //   }
    //   return property;
    // },
    viewTopDown() {
      const { Cesium, viewer } = this.cesiumInstance;
      viewer.trackedEntity = undefined;
      viewer.zoomTo(
        viewer.entities,
        new Cesium.HeadingPitchRange(0, Cesium.Math.toRadians(-90))
      );
    },
    viewSide() {
      const { Cesium, viewer } = this.cesiumInstance;
      viewer.trackedEntity = undefined;
      viewer.zoomTo(
        viewer.entities,
        new Cesium.HeadingPitchRange(
          Cesium.Math.toRadians(-90),
          Cesium.Math.toRadians(-15),
          7500
        )
      );
    },
    viewAircraft() {
      const { Cesium, viewer } = this.cesiumInstance;
      viewer.trackedEntity = this.$refs.entity.cesiumObject;
    }
  }
};
</script>

<style lang="scss" scoped>
.viewer {
  width: 100%;
  height: 700px;
  max-height: calc(100% - 50px);
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
</style>