<template>
  <div class="parallax-container" ref="container">
    <div
      v-for="(layer, index) in imageLayers"
      :key="index"
      class="parallax-layer"
      :style="getLayerStyle(index)"
    >
      <img
        :src="layer"
        :alt="`Layer ${index + 1}`"
        @load="onImageLoad(index)"
      />
    </div>
    <div v-if="loading" class="loading">Nalaganje...</div>
  </div>
</template>

<script>
export default {
  props: {
    content: { type: Object, required: true },
    imageLayers: { type: Array, required: true },
  },
  data() {
    return {
      loading: true,
      loadedImages: 0,
      gyroData: { x: 0, y: 0 },
      isSupported: false,
      permissionState: "prompt",
    };
  },
  computed: {
    totalLayers() {
      return this.imageLayers.length;
    },
  },
  mounted() {
    this.checkDeviceMotionSupport();
  },
  methods: {
    checkDeviceMotionSupport() {
      this.isSupported = window.DeviceMotionEvent !== undefined;
      if (this.isSupported) {
        if (typeof DeviceMotionEvent.requestPermission === "function") {
          this.requestPermissionManually();
        } else {
          this.startListening();
        }
      }
    },
    requestPermissionManually() {
      DeviceMotionEvent.requestPermission()
        .then((permissionState) => {
          this.permissionState = permissionState;
          if (permissionState === "granted") {
            this.startListening();
          }
        })
        .catch(console.error);
    },
    startListening() {
      window.addEventListener("devicemotion", this.handleMotion);
      this.permissionState = "granted";
    },
    handleMotion(event) {
      const { beta, gamma } = event.rotationRate || {};
      if (beta !== null && gamma !== null) {
        this.gyroData = {
          x: gamma * 0.5,
          y: beta * 0.5,
        };
      }
    },
    getLayerStyle(index) {
      const sensitivity = (index + 1) * 4;
      const translateX = this.gyroData.x * sensitivity;
      const translateY = this.gyroData.y * sensitivity;
      return {
        transform: `translate(${translateX}px, ${translateY}px)`,
        zIndex: index + 1,
      };
    },
    onImageLoad(index) {
      this.loadedImages++;
      if (this.loadedImages === this.totalLayers) {
        this.loading = false;
      }
    },
  },
  beforeUnmount() {
    window.removeEventListener("devicemotion", this.handleMotion);
  },
};
</script>

<style scoped>
.parallax-container {
  position: relative;
  width: 100%;
  height: 500px;
  overflow: hidden;
}
.parallax-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  will-change: transform;
}
.parallax-layer img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
