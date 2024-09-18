<template>
  <div v-if="imageLayers.length > 0" class="parallax-container" ref="container">
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
    <div v-if="!isSupported" class="error">
      Naprava ne podpira zaznavanja gibanja
    </div>
    <div v-if="permissionState === 'denied'" class="error">
      Dovoljenje za zaznavanje gibanja je zavrnjeno
    </div>
    <button
      v-if="needsManualPermission"
      @click="requestPermissionManually"
      class="permission-button"
    >
      Omogoči zaznavanje gibanja
    </button>
  </div>
  <div v-else class="error">Ni slik za prikaz parallax efekta</div>
</template>

<script>
export default {
  props: {
    content: { type: Object, required: true },
  },
  data() {
    return {
      loading: true,
      loadedImages: 0,
      gyroData: { x: 0, y: 0 },
      isSupported: false,
      permissionState: "prompt",
      needsManualPermission: false,
    };
  },
  computed: {
    imageLayers() {
      return this.content.imageLayers || [];
    },
    totalLayers() {
      return this.imageLayers.length;
    },
  },
  mounted() {
    console.log("Content:", this.content);
    console.log("Image Layers:", this.content.imageLayers);
    this.checkDeviceMotionSupport();
  },
  methods: {
    checkDeviceMotionSupport() {
      if (typeof window !== "undefined") {
        this.isSupported = window.DeviceMotionEvent !== undefined;
        if (this.isSupported) {
          if (typeof DeviceMotionEvent.requestPermission === "function") {
            this.needsManualPermission = true;
          } else {
            this.startListening();
          }
        }
      } else {
        this.isSupported = false;
      }
    },
    requestPermissionManually() {
      if (typeof DeviceMotionEvent.requestPermission === "function") {
        DeviceMotionEvent.requestPermission()
          .then((permissionState) => {
            this.permissionState = permissionState;
            if (permissionState === "granted") {
              this.startListening();
              this.needsManualPermission = false;
            }
          })
          .catch((error) => {
            console.error("Napaka pri zahtevanju dovoljenja:", error);
            this.permissionState = "denied";
          });
      }
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
.loading,
.error {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 10px;
  border-radius: 5px;
}
.permission-button {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>
