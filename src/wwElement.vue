<template>
  <div class="parallax-container" ref="container">
    <div class="parallax-image" :style="getImageStyle">
      <img :src="imageUrl" alt="Parallax Image" @load="onImageLoad" />
    </div>
    <div class="controls">
      <button @click="requestPermission" class="permission-button">
        {{ getPermissionButtonText }}
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
      alwaysAllow: false,
    };
  },
  computed: {
    imageUrl() {
      return "https://images.rawpixel.com/image_png_800/czNmcy1wcml2YXRlL3Jhd3BpeGVsX2ltYWdlcy93ZWJzaXRlX2NvbnRlbnQvcHUyMzMxNzg4LWltYWdlLXJtNTAzLTAxXzEtbDBqOXFyYzMucG5n.png";
    },
    getImageStyle() {
      const maxShift = 50;
      const shiftX = (this.gyroData.x / 45) * maxShift;
      const shiftY = (this.gyroData.y / 45) * maxShift;
      return {
        transform: `translate(${shiftX}px, ${shiftY}px)`,
        transition: "transform 0.1s ease-out",
      };
    },
    getPermissionButtonText() {
      if (this.permissionState === "granted") {
        return "Premikanje omogočeno";
      } else if (this.alwaysAllow) {
        return "Vedno omogoči premikanje";
      } else {
        return "Omogoči premikanje slike";
      }
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
          this.requestPermission();
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
        if (this.alwaysAllow) {
          this.grantPermission();
        } else {
          const userChoice = confirm(
            "Ali želite vedno omogočiti premikanje slike?"
          );
          if (userChoice) {
            this.alwaysAllow = true;
            this.grantPermission();
          } else {
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
          }
        }
      } else {
        this.permissionState = "granted";
        this.startListening();
      }
    },
    grantPermission() {
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
  top: -50px;
  left: -50px;
  width: calc(100% + 100px);
  height: calc(100% + 100px);
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
