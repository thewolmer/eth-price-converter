<template>
  <div class="container mx-auto px-60 mb-28">
    <h1 class="text-2xl font-bold mb-4">Ethereum Transactions</h1>
    <div class="mb-4">
      <input
        class="w-full px-4 py-2 rounded border"
        type="text"
        v-model="walletAddress"
        placeholder="Enter Ethereum wallet address"
      />
      <button class="ml-2 px-4 py-2 rounded bg-blue-500 text-white" @click="getTransactions">
        Search
      </button>
    </div>
    <div class="flex justify-between mb-4">
      <div>
        <p class="text-lg font-bold">Total Value:</p>
        <p class="text-xl font-bold">{{ totalValue.eth }} ETH</p>
        <p class="text-xl font-bold">${{ totalValue.usd }} USD</p>
      </div>
      <div>
        <p class="text-lg font-bold">Last Transaction:</p>
        <p>{{ lastTransaction.hash.slice(-6) }}</p>
        <p>{{ formatRelativeTime(lastTransaction.timeStamp) }}</p>
      </div>
    </div>
    <table class="table-auto w-full" v-if="transactions.length > 0">
      <thead>
        <tr>
          <th class="px-4 py-2">From</th>
          <th class="px-4 py-2">To</th>
          <th class="px-4 py-2">Value (ETH)</th>
          <th class="px-4 py-2">Local Time</th>
          <th class="px-4 py-2">Hash</th>
          <th class="px-4 py-2">Direction</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="transaction in reversedTransactions" :key="transaction.hash">
          <td class="border px-4 py-2">{{ transaction.from.slice(-6) }}</td>
          <td class="border px-4 py-2">{{ transaction.to.slice(-6) }}</td>
          <td class="border px-4 py-2">{{ convertWeiToEther(transaction.value) }}</td>
          <td class="border px-4 py-2">{{ formatLocalTime(transaction.timeStamp) }}</td>
          <td class="border px-4 py-2">{{ transaction.hash.slice(-6) }}</td>
          <td class="border px-4 py-2">{{ getTransactionDirection(transaction) }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>No transactions found.</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      walletAddress: '', // New data property for storing the input value
      transactions: [],
      totalValue: { eth: 0, usd: 0 },
      lastTransaction: { hash: '', timeStamp: 0 }
    }
  },
  computed: {
    reversedTransactions() {
      return this.transactions.slice().reverse() // Reverse the order of transactions
    }
  },
  async mounted() {
    await this.getTotalValue()
  },
  methods: {
    async getTransactions() {
      try {
        const response = await fetch(
          `https://api.etherscan.io/api?module=account&action=txlist&address=${this.walletAddress}&apikey=2RA7FNI57U1C6Y3P1SVWBT72EPWMWPAKJC`
        )
        const data = await response.json()
        if (data.status === '1') {
          this.transactions = data.result
        } else {
          console.error('Error retrieving transactions:', data.message)
        }
      } catch (error) {
        console.error('Error retrieving transactions:', error)
      }
    },
    async getTotalValue() {
      try {
        const response = await fetch(
          `https://api.etherscan.io/api?module=account&action=balance&address=${this.walletAddress}&tag=latest&apikey=2RA7FNI57U1C6Y3P1SVWBT72EPWMWPAKJC`
        )
        const data = await response.json()
        if (data.status === '1') {
          this.totalValue.eth = this.convertWeiToEther(data.result)
          this.totalValue.usd = this.convertEthToUsd(this.totalValue.eth)
        } else {
          console.error('Error retrieving total value:', data.message)
        }
      } catch (error) {
        console.error('Error retrieving total value:', error)
      }
    },
    convertWeiToEther(weiValue) {
      const etherValue = weiValue / Math.pow(10, 18)
      return etherValue.toFixed(6) // Display up to 6 decimal places
    },
    convertEthToUsd(ethValue) {
      // Simulated conversion rate from ETH to USD (replace with actual conversion)
      const conversionRate = 2000
      const usdValue = ethValue * conversionRate
      return usdValue.toFixed(2) // Display up to 2 decimal places
    },
    formatLocalTime(timestamp) {
      const date = new Date(parseInt(timestamp) * 1000)
      return date.toLocaleString()
    },
    getTransactionDirection(transaction) {
      if (transaction.from.toLowerCase() === this.walletAddress.toLowerCase()) {
        return 'OUT'
      } else {
        return 'IN'
      }
    },
    formatRelativeTime(timestamp) {
      const currentTime = Math.floor(Date.now() / 1000)
      const difference = currentTime - timestamp
      if (difference < 60) {
        return `${difference} seconds ago`
      } else if (difference < 3600) {
        const minutes = Math.floor(difference / 60)
        return `${minutes} minutes ago`
      } else if (difference < 86400) {
        const hours = Math.floor(difference / 3600)
        return `${hours} hours ago`
      } else {
        const days = Math.floor(difference / 86400)
        return `${days} days ago`
      }
    }
  }
}
</script>

<style scoped>
/* Your tailwindcss styles here */
</style>
