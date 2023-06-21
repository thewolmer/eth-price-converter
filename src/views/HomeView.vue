<script setup></script>

<template>
  <div class="container mx-auto h-screen flex flex-col items-center justify-center">
    <h1 class="text-2xl font-bold mb-4 text-center">ETH to USD to INR</h1>
    <div class="flex justify-center items-center">
      <input
        v-model="ethAmount"
        type="number"
        placeholder="Enter ETH amount"
        class="mr-2 px-2 py-1 border text- border-gray-300/30 rounded-md placeholder:text-gray-600/90"
      />
    </div>

    <div class="mt-9 flex justify-center items-center space-x-7">
      <div
        class="p-7 base-active rounded-lg flex-row justify-center items-center text-center shadow-lg font-bold"
      >
        <div class="bg-green-300/50 text-black inline-flex rounded-full p-2 mb-2">
          <i class="bx bx-dollar text-4xl"></i>
        </div>
        <p v-if="usdAmount">$ {{ usdAmount }}</p>
        <p v-else-if="loading" class="px-9">
          <i class="bx bx-loader-alt bx-spin text-black dark:text-white"></i>
        </p>
        <p v-else class="px-9">&nbsp;</p>
      </div>
      <div
        class="p-7 base-active rounded-lg flex-row justify-center items-center text-center shadow-lg font-bold"
      >
        <div class="bg-green-300/50 text-black inline-flex rounded-full p-2 mb-2">
          <i class="bx bx-rupee text-4xl"></i>
        </div>
        <p v-if="inrAmount">₹ {{ inrAmount }}</p>
        <p v-else-if="loading" class="px-9">
          <i class="bx bx-loader-alt bx-spin text-black dark:text-white"></i>
        </p>
        <p v-else class="px-9">&nbsp;</p>
      </div>
    </div>
  </div>
  <FooterSection />
</template>

<script>
import axios from 'axios'
import FooterSection from '../components/FooterSection.vue'

export default {
  components: { FooterSection },
  data() {
    return {
      ethAmount: '',
      usdAmount: null,
      inrAmount: null,
      loading: false
    }
  },
  watch: {
    ethAmount(value) {
      if (!value) {
        this.usdAmount = null
        this.inrAmount = null
        return
      }
      this.loading = true
      this.convertETH(value)
    }
  },
  methods: {
    async convertETH(amount) {
      try {
        this.inrAmount = ''
        this.usdAmount = ''

        const [usdResponse, inrResponse] = await Promise.all([
          axios.get('https://api.coincap.io/v2/assets/ethereum'),
          axios.get('https://api.exchangerate-api.com/v4/latest/USD')
        ])

        const { priceUsd } = usdResponse.data.data // Retrieve the priceUsd value from the response

        const { rates } = inrResponse.data
        const inrRate = rates.INR
        this.usdAmount = (parseFloat(priceUsd) * parseFloat(amount)).toFixed(2)
        this.inrAmount = (parseFloat(inrRate) * parseFloat(this.usdAmount)).toFixed(2)
        this.loading = false
      } catch (error) {
        console.error(error)
      }
    }
  }
}
</script>
