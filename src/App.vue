<template>
  <div id="app">
    <main>
      <px-header :links="links"></px-header>
      <router-view
        class="container sm:px-20 py-20 flex justify-center"
      ></router-view>
    </main>
  </div>
</template>

<script>
import PxHeader from "@/components/PxHeader";

export default {
  name: "App",
  components: {
    PxHeader,
  },
  data() {
    return {
      links: [
        {
          title: "BTC",
          to: { name: "coin-detail", params: { coin: "bitcoin" } },
        },
        {
          title: "ETH",
          to: { name: "coin-detail", params: { coin: "ethereum" } },
        },
        {
          title: "XRP",
          to: { name: "coin-detail", params: { coin: "xrp" } },
        },
        {
          title: "ALGO",
          to: { name: "coin-detail", params: { coin: "algorand" } },
        },
      ],
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
