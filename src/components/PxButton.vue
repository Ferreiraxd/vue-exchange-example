<template>
  <button
    @click="toCryto"
    class="
      bg-transparent
      hover:bg-green-500
      text-green-700
      font-semibold
      hover:text-white
      py-2
      px-2
      border border-green-500
      hover:border-transparent
      rounded
    "
  >
    <beat-loader :loading="isLoading" :color="'#68d391'" :size="8" />
    <p v-if="!isLoading">
      <slot name="content"></slot>
    </p>
  </button>
</template>

<script>
export default {
  name: "PxButton",
  props: {
    isLoading: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    toCryto() {
      console.log("me hacen click");
      this.$emit("to-crypto");
    },
  },
};
</script>

<style>
</style>