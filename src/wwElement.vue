<template>
  <div class="my-element">
    <p :style="textStyle">{{ displayText }}</p>
    <div>
      <button @click="simulateTilt('left')">Nagni levo</button>
      <button @click="simulateTilt('center')">Center</button>
      <button @click="simulateTilt('right')">Nagni desno</button>
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
    displayText() {
      return `${this.arrowDirection} (Gamma: ${this.gamma.toFixed(2)})`;
    },
  },
  methods: {
    simulateTilt(direction) {
      switch (direction) {
        case "left":
          this.gamma = -15;
          break;
        case "right":
          this.gamma = 15;
          break;
        default:
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
    margin-bottom: 10px;
  }
  button {
    margin: 0 5px;
    padding: 5px 10px;
  }
}
</style>
