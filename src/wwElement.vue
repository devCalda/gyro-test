<template>
  <div class="parallax-container" ref="container">
    <div v-if="permissionState === 'granted' && imageLayers.length > 0">
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
    </div>
    <div v-else-if="loading" class="message">Nalaganje...</div>
    <div v-else-if="!isSupported" class="message">
      Naprava ne podpira zaznavanja gibanja
    </div>
    <div v-else-if="permissionState === 'denied'" class="message">
      Dovoljenje za zaznavanje gibanja je zavrnjeno
    </div>
    <div v-else-if="imageLayers.length === 0" class="message">
      Ni slik za prikaz parallax efekta
    </div>
    <button
      v-if="permissionState === 'prompt'"
      @click="requestPermission"
      class="permission-button"
    >
      Začni demonstracijo
    </button>
    <div v-if="permissionState === 'granted'" class="sensor-data">
      <p>Beta (X-os): {{ gyroData.y.toFixed(2) }}°</p>
      <p>Gamma (Y-os): {{ gyroData.x.toFixed(2) }}°</p>
    </div>
  </div>
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
      if (typeof window !== "undefined" && window.DeviceOrientationEvent) {
        this.isSupported = true;
      } else {
        this.isSupported = false;
      }
    },
    requestPermission() {
      if (typeof DeviceOrientationEvent.requestPermission === "function") {
        DeviceOrientationEvent.requestPermission()
          .then((permissionState) => {
            this.permissionState = permissionState;
            if (permissionState === "granted") {
              this.startListening();
            }
          })
          .catch((error) => {
            console.error("Napaka pri zahtevanju dovoljenja:", error);
            this.permissionState = "denied";
          });
      } else {
        this.permissionState = "granted";
        this.startListening();
      }
    },
    startListening() {
      window.addEventListener("deviceorientation", this.handleOrientation);
    },
    handleOrientation(event) {
      this.gyroData = {
        x: event.gamma || 0,
        y: event.beta || 0,
      };
    },
    getLayerStyle(index) {
      const sensitivity = (index + 1) * 0.5;
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
    window.removeEventListener("deviceorientation", this.handleOrientation);
  },
};
</script>

<style scoped>
.parallax-container {
  position: relative;
  width: 100%;
  height: 500px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
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
.message {
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 10px;
  border-radius: 5px;
  text-align: center;
}
.permission-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.sensor-data {
  position: absolute;
  bottom: 10px;
  left: 10px;
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 10px;
  border-radius: 5px;
}
</style>
