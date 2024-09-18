<template>
  <div class="my-element">
    <p :style="textStyle">{{ displayText }}</p>
    <button
      v-if="!isSupported || permissionState !== 'granted'"
      @click="requestPermission"
    >
      Omogoči zaznavanje gibanja
    </button>
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
        return "Potrebno dovoljenje za zaznavanje gibanja";
      return `${this.arrowDirection} (Gamma: ${this.gamma.toFixed(2)})`;
    },
  },
  mounted() {
    this.checkDeviceMotionSupport();
  },
  methods: {
    checkDeviceMotionSupport() {
      this.isSupported = window.DeviceMotionEvent !== undefined;
      if (
        this.isSupported &&
        typeof DeviceMotionEvent.requestPermission === "function"
      ) {
        this.permissionState = "prompt";
      } else if (this.isSupported) {
        this.permissionState = "granted";
        this.startListening();
      }
    },
    requestPermission() {
      if (typeof DeviceMotionEvent.requestPermission === "function") {
        DeviceMotionEvent.requestPermission()
          .then((permissionState) => {
            this.permissionState = permissionState;
            if (permissionState === "granted") {
              this.startListening();
            }
          })
          .catch(console.error);
      } else {
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
  button {
    padding: 5px 10px;
    margin-top: 10px;
  }
}
</style>
