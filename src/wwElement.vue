<template>
  <div class="my-element">
    <p :style="textStyle">{{ arrowDirection }}</p>
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
  },
  mounted() {
    if (typeof window !== "undefined" && window.DeviceMotionEvent) {
      window.addEventListener("devicemotion", this.handleMotion);
    }
  },
  beforeUnmount() {
    if (typeof window !== "undefined" && window.DeviceMotionEvent) {
      window.removeEventListener("devicemotion", this.handleMotion);
    }
  },
  methods: {
    handleMotion(event) {
      if (event.rotationRate) {
        this.gamma = event.rotationRate.gamma;
      }
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
