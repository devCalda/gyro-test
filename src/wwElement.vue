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
    window.addEventListener("keydown", this.handleKeyPress);
  },
  beforeUnmount() {
    window.removeEventListener("keydown", this.handleKeyPress);
  },
  methods: {
    handleKeyPress(event) {
      if (event.key === "ArrowLeft") {
        this.gamma = -15;
      } else if (event.key === "ArrowRight") {
        this.gamma = 15;
      } else {
        this.gamma = 0;
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
