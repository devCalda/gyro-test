<template>
  <div class="my-element">
    <p :style="textStyle">{{ displayText }}</p>
  </div>
</template>

<script>
export default {
  props: {
    content: { type: Object, required: true },
  },
  data() {
    return {
      gamma: 0,
      isSupported: false,
      permissionState: "prompt",
    };
  },
  computed: {
    textStyle() {
      return {
        color: this.content.textColor,
      };
    },
    arrowDirection() {
      if (!this.isSupported || this.permissionState !== "granted")
        return "Potrebno dovoljenje";
      if (this.gamma > 10) return "→";
      if (this.gamma < -10) return "←";
      return "I am a custom element !";
    },
    displayText() {
      if (!this.isSupported) return "Naprava ne podpira zaznavanja gibanja";
      if (this.permissionState !== "granted")
        return "Zahtevam dovoljenje za zaznavanje gibanja...";
      return `${this.arrowDirection} (Gamma: ${this.gamma.toFixed(2)})`;
    },
  },
  mounted() {
    this.checkDeviceMotionSupport();
  },
  methods: {
    checkDeviceMotionSupport() {
      this.isSupported = window.DeviceMotionEvent !== undefined;
      if (this.isSupported) {
        this.requestPermissionAndStart();
      }
    },
    requestPermissionAndStart() {
      if (typeof DeviceMotionEvent.requestPermission === "function") {
        DeviceMotionEvent.requestPermission()
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
        // Za naprave, ki ne zahtevajo eksplicitnega dovoljenja
        this.permissionState = "granted";
        this.startListening();
      }
    },
    startListening() {
      window.addEventListener("devicemotion", this.handleMotion);
    },
    handleMotion(event) {
      if (event.rotationRate) {
        this.gamma = event.rotationRate.gamma || 0;
      }
    },
  },
  beforeUnmount() {
    window.removeEventListener("devicemotion", this.handleMotion);
  },
};
</script>

<style lang="scss" scoped>
.my-element {
  p {
    font-size: 18px;
    margin-bottom: 10px;
  }
}
</style>
