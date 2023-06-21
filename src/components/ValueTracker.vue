<script setup>
import { useDark, useToggle } from '@vueuse/core'
const isDark = useDark()
const toggleDark = useToggle(isDark)
</script>

<template>
  <section class="bottom-0 fixed w-full base-color text-sm font-mono">
    <div class="flex justify-center md:justify-between">
      <div class="flex items-center space-x-2 ml-4">
        <div class="flex items-center space-x-2 border px-4 border-dark-text-idle">
          <i
            :class="[
              loading
                ? 'text-yellow-500 bx bx-loader bx-spin text-lg'
                : 'bx bx-radio-circle-marked bx-burst text-lg text-green-500'
            ]"
          ></i>
          <p
            :class="[loading ? 'text-yellow-500 text-xs animate-pulse ' : 'text-gray-500 text-xs']"
          >
            {{ currentLocalTime }}
          </p>
        </div>

        <div class="flex items-center border px-4 border-dark-text-idle">
          <!-- <img src="src/assets/eth-logo.svg" class="w-2" alt="ETH Logo" /> -->
          <h2 :class="[ethChange < 0 ? 'text-red-400 font-bold' : 'text-green-500 font-bold']">
            ETH : ${{ ethValue }} ({{ ethChange }}%)
          </h2>
        </div>
      </div>
      <div class="md:flex items-center space-x-4 mr-4 hidden">
        <h2><i class="bx bxs-gas-pump text-xl"></i></h2>
        <div class="gas-item border px-4 border-dark-text-idle">
          <h3>
            Safe: {{ gasPrices.safeLow }} Gwei | Average: {{ gasPrices.average }} Gwei | Fast:
            {{ gasPrices.fast }} Gwei
          </h3>
        </div>
        <button
          @click="toggleDark()"
          class="flex overflow-hidden items-center p-1 base-hover rounded-full"
        >
          <div v-if="isDark == false" class="flex">
            <i class="bx bx-moon text-xl text-light-heading"></i>
          </div>
          <div v-else class="flex">
            <i class="bx bx-sun text-xl text-dark-heading"></i>
          </div>
        </button>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      ethValue: null,
      ethChange: null,
      gasPrices: {
        safeLow: null,
        average: null,
        fast: null,
        currentLocalTime: null
      },
      lastUpdate: null,
      loading: false
    }
  },
  created() {
    this.fetchEthValue()
    this.fetchGasPrice()
    setInterval(this.fetchEthValue, 5000) // Fetch Ethereum value every 5 seconds
    setInterval(this.fetchGasPrice, 5000) // Fetch gas price every 5 seconds
  },
  methods: {
    async fetchEthValue() {
      try {
        this.loading = true
        const response = await axios.get(
          'https://api.coincap.io/v2/assets/ethereum?utm_source=wolmer.me'
        )
        const { priceUsd, changePercent24Hr } = response.data.data

        this.ethValue = parseFloat(priceUsd).toFixed(2)
        this.ethChange = parseFloat(changePercent24Hr).toFixed(2)
        this.loading = false
      } catch (error) {
        console.error(error)
      }
    },
    async fetchGasPrice() {
      try {
        const response = await axios.get(
          'https://ethgasstation.info/api/ethgasAPI.json?utm_source=wolmer.me'
        )
        const { safeLow, average, fast } = response.data

        this.gasPrices.safeLow = safeLow / 10
        this.gasPrices.average = average / 10
        this.gasPrices.fast = fast / 10
        this.lastUpdate = new Date().toLocaleTimeString()

        const currentTime = new Date().toLocaleTimeString()
        this.currentLocalTime = currentTime
      } catch (error) {
        console.error(error)
      }
    }
  }
}
</script>

<style scoped>
.gas-info {
  margin-top: 20px;
}

.eth-value {
  margin-bottom: 20px;
}

.negative {
  color: red;
}

.positive {
  color: green;
}
</style>
