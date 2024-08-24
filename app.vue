<template>
  <div>
    <NuxtRouteAnnouncer />
    <!-- <NuxtWelcome /> -->
    <div class="flex items-center justify-center gap-2 min-h-dvh bg-gray-300">
      <button class="bg-black text-white p-2" @click="getBTCPrice">Get Btc Price</button>
      <span>
        {{ btcPrice }} BTC
      </span>
      <button class="bg-black text-white p-2" @click="getETHPrice">Get Eth Price</button>
      <span>
        {{ ethPrice }} ETH
      </span>
    </div>
  </div>
</template>

<script setup>
  import { Web3 } from "web3";
  import { ChainlinkPlugin, MainnetPriceFeeds } from "@chainsafe/web3-plugin-chainlink";

  const btcPrice = ref("000");
  const ethPrice = ref("000");

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

</script>
