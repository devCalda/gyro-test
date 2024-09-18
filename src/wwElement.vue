<template>
  <div class="parallax-container" ref="container">
    <div class="parallax-image" :style="getImageStyle">
      <img :src="imageUrl" alt="Parallax Image" @load="onImageLoad" />
    </div>
    <div class="controls">
      <button @click="requestPermission" class="permission-button">
        {{
          permissionState === "granted"
            ? "Premikanje omogočeno"
            : "Omogoči premikanje slike"
        }}
      </button>
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
      const maxShift = 50; // Povečamo največji premik na 50 pikslov
      const shiftX = (this.gyroData.x / 45) * maxShift; // Povečamo občutljivost
      const shiftY = (this.gyroData.y / 45) * maxShift; // Povečamo občutljivost
      return {
        transform: `translate(${shiftX}px, ${shiftY}px)`,
        transition: "transform 0.1s ease-out",
      };
    },
  },
  mounted() {
    this.checkDeviceMotionSupport();
  },
  methods: {
    checkDeviceMotionSupport() {
      if (window.DeviceOrientationEvent) {
        this.isSupported = true;
        if (typeof DeviceOrientationEvent.requestPermission === "function") {
          this.requestPermission(); // Poskusimo samodejno zahtevati dovoljenje
        } else {
          this.permissionState = "granted";
          this.startListening();
        }
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
  height: 500px;
  overflow: hidden;
}
.parallax-image {
  position: absolute;
  top: -50px; /* Dodamo negativni odmik zgoraj */
  left: -50px; /* Dodamo negativni odmik levo */
  width: calc(100% + 100px); /* Povečamo širino za 100px */
  height: calc(100% + 100px); /* Povečamo višino za 100px */
  will-change: transform;
}
.parallax-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.controls {
  position: absolute;
  bottom: 20px;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: center;
  z-index: 10;
}
.permission-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>
