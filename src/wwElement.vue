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
import { useDeviceMotion, usePermission } from "@vueuse/core";
import { computed, ref, watch } from "vue";

export default {
  props: {
    content: { type: Object, required: true },
  },
  setup() {
    const { gamma, isSupported } = useDeviceMotion();
    const motionPermission = usePermission("accelerometer");
    const permissionState = ref(motionPermission.state);

    const arrowDirection = computed(() => {
      if (!isSupported || permissionState.value !== "granted")
        return "Potrebno dovoljenje";
      if (gamma.value > 10) return "→";
      if (gamma.value < -10) return "←";
      return "I am a custom element !";
    });

    const displayText = computed(() => {
      if (!isSupported) return "Naprava ne podpira zaznavanja gibanja";
      if (permissionState.value !== "granted")
        return "Potrebno dovoljenje za zaznavanje gibanja";
      return `${arrowDirection.value} (Gamma: ${
        gamma.value?.toFixed(2) ?? "N/A"
      })`;
    });

    const requestPermission = async () => {
      if (typeof DeviceMotionEvent.requestPermission === "function") {
        const permission = await DeviceMotionEvent.requestPermission();
        permissionState.value = permission;
      } else {
        permissionState.value = "granted";
      }
    };

    watch(motionPermission, (newState) => {
      permissionState.value = newState;
    });

    return { displayText, isSupported, permissionState, requestPermission };
  },
  computed: {
    textStyle() {
      return {
        color: this.content.textColor,
      };
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
    padding: 5px 10px;
    margin-top: 10px;
  }
}
</style>
