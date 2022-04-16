<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <bounce-loader :loading="isLoading" :color="'#68d391'" :size="100" />
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            class="w-20 h-20 mr-5"
            :src="`https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`"
            :alt="asset.name"
          />
          <h1 class="text-5xl font-bold">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span>{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-blue-600 mr-10 uppercase">Precio actual</b>
              <span class="text-blue-600">{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-red-600 mr-10 uppercase">Precio más bajo</b>
              <span class="text-red-600">{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-green-600 mr-10 uppercase">Precio más alto</b>
              <span class="text-green-600">{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio Promedio</b>
              <span>{{ average | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span
                :class="
                  Number(asset.changePercent24Hr) < 0
                    ? 'text-red-600'
                    : 'text-green-600'
                "
                >{{ asset.changePercent24Hr | percent }}</span
              >
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            @click="toggleConverter"
            class="
              bg-green-500
              hover:bg-green-700
              text-white
              font-bold
              py-2
              px-4
              rounded
            "
          >
            {{ buttonConvertText }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="convertValue"
                id="convertValue"
                type="number"
                class="
                  text-center
                  bg-white
                  focus:outline-none focus:shadow-outline
                  border border-gray-300
                  rounded-lg
                  py-2
                  px-4
                  block
                  w-full
                  appearance-none
                  leading-normal
                "
                :placeholder="`Valor en ${fromUsd ? 'USD' : asset.symbol}`"
              />
            </label>
          </div>

          <span class="text-xl"
            >{{ convertResult }} {{ fromUsd ? asset.symbol : "USD" }}</span
          >
        </div>
      </div>

      <trend
        :data="setChartTrendData"
        :gradient="['#00ff00', '#ff0000']"
        :min="min"
        :max="max"
        auto-draw
        smooth
      >
      </trend>

      <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
      <table>
        <tr
          v-for="m in markets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
          class="border-b"
        >
          <td>
            <b>{{ m.exchangeId }}</b>
          </td>
          <td>{{ m.priceUsd | dollar }}</td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button
              :is-loading="m.isLoading || false"
              v-if="!m.url"
              @to-crypto="getWebsite(m)"
            >
              <template v-slot:content>
                <span>Obtener Url</span>
              </template>
            </px-button>
            <a v-else class="hover:underline text-green-600" target="_blanck">
              {{ m.url }}</a
            >
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import api from "@/api";
import PxButton from "@/components/PxButton.vue";

export default {
  components: { PxButton },
  props: {
    coin: {
      type: String,
      default: () => "bitcoin",
    },
  },
  name: "CoinDetail",
  data() {
    return {
      isLoading: false,
      asset: {},
      history: [],
      markets: [],
      fromUsd: true,
      convertValue: null,
    };
  },

  watch: {
    $route: function () {
      this.getCoinData();
    },
  },

  created() {
    this.getCoinData();
    //await api.getAsset(this.coin).then(res => this.asset = res);
  },
  methods: {
    /*async getCoinData(){
        await api.getAsset(this.coin).then(res => this.asset = res);
      }*/
    toggleConverter() {
      this.fromUsd = !this.fromUsd;
    },

    getWebsite(exchange) {
      this.$set(exchange, "isLoading", true);
      return api
        .getExchange(exchange.exchangeId)
        .then((res) => {
          console.log(res);
          this.$set(exchange, "url", res.exchangeUrl);
        })
        .finally(() => {
          this.$set(exchange, "isLoading", false);
        });
    },

    getCoinData() {
      this.isLoading = true;
      Promise.all([
        api.getAsset(this.coin),
        api.getAssetHistory(this.coin),
        api.getMarkets(this.coin),
      ])
        .then(([asset, history, markets]) => {
          this.asset = asset;
          this.history = history;
          this.markets = markets;
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
  },
  computed: {
    buttonConvertText() {
      if (this.fromUsd === true) {
        return `USD to ${this.asset.symbol}`;
      } else {
        return `${this.asset.symbol} to USD`;
      }
    },
    min() {
      return Math.min(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      );
    },

    max() {
      return Math.max(
        ...this.history.map((h) => parseFloat(h.priceUsd).toFixed(2))
      );
    },
    average() {
      return Math.abs(
        ...this.history.map((hour) => {
          return parseFloat(hour.priceUsd);
        })
      );
    },
    setChartTrendData() {
      console.log(
        this.history.map((hours) =>
          parseInt(Number(hours.priceUsd).toFixed(2), 10)
        )
      );
      return this.history.map((hours) =>
        parseInt(Number(hours.priceUsd).toFixed(2), 10)
      );
    },
    chartData() {
      const data = [];
      this.history.map((h) => {
        data.push([h.date, parseFloat(h.priceUsd).toFixed(2)]);
      });
      return data;
    },
    convertResult() {
      if (!this.convertValue) {
        return 0;
      }

      const result = this.fromUsd
        ? this.convertValue / this.asset.priceUsd
        : this.convertValue * this.asset.priceUsd;

      return result.toFixed(4);
    },
  },
};
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>

