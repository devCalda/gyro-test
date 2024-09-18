<template>
  <div class="parallax-container" ref="container">
    <div v-if="permissionState === 'granted' && imageUrl">
      <div class="parallax-image" :style="getImageStyle">
        <img :src="imageUrl" alt="Parallax Image" @load="onImageLoad" />
      </div>
    </div>
    <div v-else-if="loading" class="message">Nalaganje...</div>
    <div v-else-if="!isSupported" class="message">
      Naprava ne podpira zaznavanja gibanja
    </div>
    <div v-else-if="permissionState === 'denied'" class="message">
      Dovoljenje za zaznavanje gibanja je zavrnjeno
    </div>
    <div v-else-if="!imageUrl" class="message">
      Ni slike za prikaz parallax efekta
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
      gyroData: { x: 0, y: 0 },
      isSupported: false,
      permissionState: "prompt",
    };
  },
  computed: {
    imageUrl() {
      return this.content.imageUrl || "";
    },
    getImageStyle() {
      const maxShift = 20; // Največji premik v pikslih
      const shiftX = (this.gyroData.x / 90) * maxShift;
      const shiftY = (this.gyroData.y / 90) * maxShift;
      return {
        transform: `translate(${shiftX}px, ${shiftY}px)`,
        transition: "transform 0.1s ease-out",
      };
    },
  },
  mounted() {
    console.log("Content:", this.content);
    console.log("Image URL:", this.imageUrl);
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
        x: event.gamma || 0, // Nagib levo-desno
        y: event.beta || 0, // Nagib naprej-nazaj
      };
    },
    onImageLoad() {
      this.loading = false;
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
  height: 300px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
}
.parallax-image {
  position: absolute;
  top: -10%;
  left: -10%;
  width: 120%;
  height: 120%;
  will-change: transform;
}
.parallax-image img {
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
