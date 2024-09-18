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
      eventCount: 0,
      lastEventTime: null,
    };
  },
  computed: {
    textStyle() {
      return {
        color: this.content.textColor,
      };
    },
    arrowDirection() {
      if (this.gamma > 10) return "→";
      if (this.gamma < -10) return "←";
      return "I am a custom element !";
    },
    displayText() {
      return `${this.arrowDirection} (Gamma: ${this.gamma.toFixed(
        2
      )}, Events: ${this.eventCount})`;
    },
  },
  mounted() {
    this.setupDeviceMotion();
  },
  beforeUnmount() {
    this.removeDeviceMotion();
  },
  methods: {
    setupDeviceMotion() {
      if (typeof window !== "undefined") {
        if (window.DeviceMotionEvent) {
          if (typeof DeviceMotionEvent.requestPermission === "function") {
            // iOS 13+ zahteva dovoljenje
            DeviceMotionEvent.requestPermission()
              .then((permissionState) => {
                if (permissionState === "granted") {
                  window.addEventListener("devicemotion", this.handleMotion);
                } else {
                  console.log("Dovoljenje za zaznavanje gibanja zavrnjeno");
                }
              })
              .catch(console.error);
          } else {
            // Starejše naprave ali Android
            window.addEventListener("devicemotion", this.handleMotion);
          }
        } else {
          console.log("DeviceMotionEvent ni podprt na tej napravi");
        }
      }
    },
    removeDeviceMotion() {
      if (typeof window !== "undefined" && window.DeviceMotionEvent) {
        window.removeEventListener("devicemotion", this.handleMotion);
      }
    },
    handleMotion(event) {
      this.eventCount++;
      this.lastEventTime = new Date().toISOString();
      if (event.rotationRate) {
        this.gamma = event.rotationRate.gamma || 0;
      } else if (event.accelerationIncludingGravity) {
        // Alternativni način za starejše naprave
        this.gamma = event.accelerationIncludingGravity.y || 0;
      }
      console.log(
        `Dogodek: ${this.eventCount}, Čas: ${this.lastEventTime}, Gamma: ${this.gamma}`
      );
    },
  },
};
</script>

<style lang="scss" scoped>
.my-element {
  p {
    font-size: 18px;
  }
}
</style>
