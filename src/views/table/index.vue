<template>
  <div class="viewer">
    <vc-viewer @ready="ready">
      <vc-primitive-polyline-ground :appearance="appearance">
        <vc-instance-geometry>
          <vc-geometry-polyline-ground ref="polylineGround" :positions="positions" :width="10"></vc-geometry-polyline-ground>
        </vc-instance-geometry>
      </vc-primitive-polyline-ground>
      <vc-provider-terrain-cesium></vc-provider-terrain-cesium>
      <vc-layer-imagery>
        <vc-provider-imagery-mapbox mapId="mapbox.streets"></vc-provider-imagery-mapbox>
      </vc-layer-imagery>
    </vc-viewer>
  </div>
</template>
<script>
  export default {
   data() {
      return {
        appearance: {},
        geometryInstances: {},
        positions: [
          { lng: 100.1340164450331, lat: 31.05494287836128 },
          { lng: 108.08821010582645, lat: 31.05494287836128 },
          { lng: 108.07821010582645, lat: 31.05494287836128 }
        ]
      }
    },
    mounted() {
      this.$refs.polylineGround.createPromise.then(({ Cesium, viewer, cesiumObject }) => {
        const boundingSphere = Cesium.BoundingSphere.fromPoints(cesiumObject._positions)
        viewer.scene.camera.flyToBoundingSphere(boundingSphere)
      })
    },
    methods: {
      ready({ Cesium, viewer }) {
        this.appearance = new Cesium.PolylineMaterialAppearance()
      }
    }
  }
</script>

<style lang="scss" scoped>
.viewer{
  width:100%;
  height:700px;
  max-height: calc(100% - 50px);
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