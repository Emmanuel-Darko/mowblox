<template>
  <div>
    <NuxtRouteAnnouncer />
    <!-- <NuxtWelcome /> -->
    <div class="flex flex-col items-center justify-center gap-2 min-h-dvh bg-gray-300 flex-wrap">
      <img src="https://mowblox.com/_next/image?url=%2Fassets%2Fmowblox_logo_lg.png&w=256&q=75" alt="logo">
      <h1 class="font- font-bold mb-3">BTC/ETH Converter</h1>

      <div class="flex justify-center items-center gap-5 mb-5">
        <button @click="currentView='btc'" class="bg-gray-400 p-4 rounded">BTC</button>
        <button @click="currentView='eth'" class="bg-gray-400 p-4 rounded">ETH</button>
      </div>
      <div class="bg-blue-300 p-2 rounded-md mb-5">
        <span>
          Current BTC = {{ btcPrice }}
        </span>
        <span>
          Current ETH  = {{ ethPrice }}
        </span>
      </div>

      <div v-if="currentView == 'btc'" class="">
        <h3>Enter BTC Price</h3>
        <input v-model="btc" type="number" class="p-3">

        <h3>USD</h3>
        <input v-model="btcUsd" type="number" class="p-3"/>
      </div>
      
      <div v-if="currentView == 'eth'" class="">
        <h3>Enter ETH Price</h3>
        <input v-model="eth" type="number" class="p-3 mb-5">

        <h3>USD</h3>
        <input v-model="ethUsd" type="number" class="p-3"/>
      </div>

    </div>
  </div>
</template>

<script setup>
  import { Web3 } from "web3";
  import { ChainlinkPlugin, MainnetPriceFeeds } from "@chainsafe/web3-plugin-chainlink";
  const currentView = ref('btc')

  const btcPrice = ref("000");
  const ethPrice = ref("000");

  const btc = ref(null)
  const eth = ref(null)
  const btcUsd = computed(() => (btc.value * btcPrice.value)?.toFixed(2)?.toString())
  const ethUsd = computed(() => (eth.value * ethPrice.value)?.toFixed(2)?.toString())

  // Initialize RPC/injected provider
  const web3 = new Web3();

  // Register the plugin
  web3.registerPlugin(new ChainlinkPlugin());

  async function getBTCPrice() {
    // use plugin
    //calling the plugin
    const btcpriceLocal = await web3.chainlink.getPrice(MainnetPriceFeeds.BtcUsd);
    //formating the variable
    const formattedPrice = btcpriceLocal.answer.toString().substring(0, 5);
    //updating front end
    btcPrice.value = formattedPrice;
  }

  async function getETHPrice() {
    // use plugin
    //calling the plugin
    const ethPriceLocal = await web3.chainlink.getPrice(MainnetPriceFeeds.EthUsd);
    //formating the variable
    const formattedPrice = ethPriceLocal.answer.toString().substring(0, 4);
    //updating front end
    ethPrice.value = formattedPrice;
  }

  onMounted(async() => {
    await getBTCPrice()
    await getETHPrice()
  })

</script>
