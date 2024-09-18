<template>
  <div class="parallax-container" ref="container">
    <div class="parallax-image" :style="getImageStyle">
      <img :src="imageUrl" alt="Parallax Image" @load="onImageLoad" />
    </div>
    <button
      v-if="permissionState === 'prompt'"
      @click="requestPermission"
      class="permission-button"
    >
      Omogoči premikanje slike
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
      return "https://images.rawpixel.com/image_png_800/czNmcy1wcml2YXRlL3Jhd3BpeGVsX2ltYWdlcy93ZWJzaXRlX2NvbnRlbnQvcHUyMzMxNzg4LWltYWdlLXJtNTAzLTAxXzEtbDBqOXFyYzMucG5n.png";
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
    console.log("Component mounted");
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
      console.log("Slika naložena");
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
}
.parallax-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  will-change: transform;
}
.parallax-image img {
  width: 100%;
  height: 100%;
  object-fit: contain; /* Spremenimo v 'contain', da ohranimo razmerje slike */
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
