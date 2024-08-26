<template>
  <div class="min-h-dvh bg-gray-300 flex-wrap p-5">
    <NuxtRouteAnnouncer />
    <header class="flex justify-between items-center">
      <img class="w-32 sm:w-[180px]" src="https://mowblox.com/_next/image?url=%2Fassets%2Fmowblox_logo_lg.png&w=256&q=75" alt="logo">
      <div>
        <button 
          v-if="!userAddress" 
          @click="loginWithMetaMask" 
          :disabled="loading" 
          class="bg-black text-white p-2 rounded-lg"
        >
          <span v-if="loading">Connecting...</span>
          <span v-else class="flex justify-center items-center gap-2">
            Connect         
            <Icon name="ion:wallet-outline" size="30" class="text-center" />          
          </span>
        </button>
        
        <div v-else
          class="flex justify-center items-center gap-2"
        >
          <span class="p-1 rounded bg-[#f7941a] text-white shadow">{{ truncatedAddress }}</span>
          <button @click="logout" class="bg-black text-white p-2 rounded-lg">
            Logout
          </button>
        </div>
      </div>
    </header>


    <div class="flex flex-col items-center justify-center gap-10">
      <h1 class="mt-5 font- font-bold mb-3 text-2xl">Mowblox Coin Converter</h1>

      <div class="flex justify-center items-center gap-5">
        <button @click="currentView='btc'" 
          class="bg-gray-200 p-4 rounded flex justify-center items-center gap-2"
          :class="[{'border-2 border-[#f7941a7e] shadow-2xl': currentView=='btc'}, {'opacity-80': currentView!=='btc'}]"
        >
          <img class="w-10" src="https://dynamic-assets.coinbase.com/e785e0181f1a23a30d9476038d9be91e9f6c63959b538eabbc51a1abc8898940383291eede695c3b8dfaa1829a9b57f5a2d0a16b0523580346c6b8fab67af14b/asset_icons/b57ac673f06a4b0338a596817eb0a50ce16e2059f327dc117744449a47915cb2.png" alt="btc">
          <span :class="{'font-bold': currentView=='btc'}">BTC</span>
        </button>

        <button @click="currentView='eth'" 
          class="bg-gray-200 p-4 rounded flex justify-center items-center gap-2"
          :class="[{'border-2 border-[#627EEA] shadow-2xl': currentView=='eth'}, {'opacity-80': currentView!=='eth'}]"
        >
          <img class="w-10" src="https://dynamic-assets.coinbase.com/dbb4b4983bde81309ddab83eb598358eb44375b930b94687ebe38bc22e52c3b2125258ffb8477a5ef22e33d6bd72e32a506c391caa13af64c00e46613c3e5806/asset_icons/4113b082d21cc5fab17fc8f2d19fb996165bcce635e6900f7fc2d57c4ef33ae9.png" alt="btc">
          <span :class="{'font-bold': currentView=='eth'}">ETH</span>
        </button>

        <button @click="currentView='sol'" 
          class="bg-gray-200 p-4 rounded flex justify-center items-center gap-2"
          :class="[{'border-2 border-[#000000e3] shadow-2xl': currentView=='sol'}, {'opacity-80': currentView!=='sol'}]"
        >
          <img class="w-10" src="https://asset-metadata-service-production.s3.amazonaws.com/asset_icons/b658adaf7913c1513c8d120bcb41934a5a4bf09b6adbcb436085e2fbf6eb128c.png" alt="btc">
          <span :class="{'font-bold': currentView=='sol'}">SOL</span>
        </button>
      </div>
      
      <!-- Current Price -->
      <div 
        class="p-2 rounded-md text-white font-bold"
        :class="[{'bg-[#f7941ae7]': currentView=='btc'}, {'bg-[#627EEA]': currentView=='eth'}, {'bg-[#000000]': currentView=='sol'}]"
      >
        <span v-if="currentView=='btc'">
          Current BTC = $ {{ btcPrice?.toLocaleString() }}
        </span>
        <span v-if="currentView=='eth'">
          Current ETH  = $ {{ ethPrice }}
        </span>
        <span v-if="currentView=='sol'">
          Current SOL  = $ {{ solPrice }}
        </span>
      </div>

      <!-- Bitcoin -->
      <div v-if="currentView == 'btc'" class="flex flex-col gap-2 justify-center items-center p-10 bg-gray-200 border-2 border-dotted border-[#f7941a] rounded shadow">
        <div class="flex flex-row justify-center items-center gap-4">
          <h3 class="font-bold">BTC</h3>
          <input v-model="computedBtc" type="number" placeholder="Eg. 1" 
            class="p-3 rounded-lg outline-[#f7941ae7]"
            @input="(e) => generateUsd(e, 'btc')"
          >
        </div>
        <Icon name="heroicons-arrows-up-down" size="30" class="text-center bg-[#00000097]" />
        <div class="flex flex-row justify-center items-center gap-4">
          <h3 class="font-bold">USD($)</h3>
          <input v-model="computedBtcUsd" @input="(e) => convertBack(e, 'btc')" type="number" placeholder="Eg. 1" 
            class="p-3 rounded-lg outline-[#f7941ae7]"
          >
        </div>
      </div>
      
      <!-- Ether -->
      <div v-if="currentView == 'eth'" class="flex flex-col gap-2 justify-center items-center p-10 bg-gray-200 border-2 border-dotted border-[#627EEA] rounded shadow">
        <div class="flex flex-row justify-center items-center gap-4">
          <h3 class="font-bold">ETH</h3>
          <input v-model="computedEth" type="number" placeholder="Eg. 1" 
            class="p-3 rounded-lg outline-[#627EEA]"
            @input="(e) => generateUsd(e, 'eth')"
          >
        </div>
        <Icon name="heroicons-arrows-up-down" size="30" class="text-center bg-[#00000097]" />
        <div class="flex flex-row justify-center items-center gap-4">
          <h3 class="font-bold">USD($)</h3>
          <input v-model="computedEthUsd" type="number" placeholder="Eg. 1" 
            class="p-3 rounded-lg outline-[#627EEA]"
            @input="(e) => convertBack(e, 'eth')"
          >
        </div>
      </div>

      <!-- Solana -->
      <div v-if="currentView == 'sol'" class="flex flex-col gap-2 justify-center items-center p-10 bg-gray-200 border-2 border-dotted border-[#00000097] rounded shadow">
        <div class="flex flex-row justify-center items-center gap-4">
          <h3 class="font-bold">SOL</h3>
          <input v-model="computedSol" type="number" placeholder="Eg. 1" 
            class="p-3 rounded-lg outline-[#00000097]"
            @input="(e) => generateUsd(e, 'sol')"
          >
        </div>
        <Icon name="heroicons-arrows-up-down" size="30" class="text-center bg-[#00000097]" />
        <div class="flex flex-row justify-center items-center gap-4">
          <h3 class="font-bold">USD($)</h3>
          <input v-model="computedSolUsd" type="number" placeholder="Eg. 1" 
            class="p-3 rounded-lg outline-[#00000097]"
            @input="(e) => convertBack(e, 'sol')"
          >
        </div>
      </div>
    </div>
    <img class="hidden sm:block w-40 absolute right-0 opacity-50" src="https://docs.web3js.org/img/web3js.svg" alt="web3logo">
    <img class="hidden sm:block w-40 absolute left-40 top-40 opacity-50" src="https://docs.web3js.org/img/web3js.svg" alt="web3logo">
    <img class="hidden sm:block w-40 absolute left-0 opacity-50" src="https://docs.web3js.org/img/web3js.svg" alt="web3logo">
    <img class="hidden sm:block w-40 absolute right-20 top-40 opacity-50" src="https://docs.web3js.org/img/web3js.svg" alt="web3logo">
    <footer class="place-self-center w-100 absolute bottom-0 sm:right-0 p-5 flex justify-between items-center">
      <div class="flex justify-center items-center">
        <span> Crafted with &MediumSpace;</span>
        <Icon name="ion:heart" size="25" class="text-center animate-bounce" style="color: red" />
        <span> &MediumSpace; by <b>Emmanuel Darko</b></span>
      </div>
      <NuxtLink to="https://github.com/Emmanuel-Darko/mowblox" target="_blank">
        &MediumSpace;
        <Icon name="ion:logo-github" size="30" class="text-center" />
      </NuxtLink>
    </footer>
  </div>
</template>

<script setup>
  import { Web3 } from "web3";
  import { ChainlinkPlugin, MainnetPriceFeeds } from "@chainsafe/web3-plugin-chainlink";
  const currentView = ref('btc')

  const btcPrice = ref("000");
  const ethPrice = ref("000");
  const solPrice = ref("000");
  const btc = ref(null)
  const eth = ref(null)
  const sol = ref(null)

  const btcUsd = ref(null)
  const ethUsd = ref(null)
  const solUsd = ref(null)

  const userAddress = ref("")
  const loading = ref(false)
  const token = useCookie('token')


  // btc
  const computedBtc = computed(() => btc.value)
  const computedBtcUsd = computed(() => btcUsd.value)
  // eth
  const computedEth = computed(() => eth.value)
  const computedEthUsd = computed(() => ethUsd.value)
  // sol
  const computedSol = computed(() => sol.value)
  const computedSolUsd = computed(() => solUsd.value)

  const generateUsd = (event, coin) => {
    if(coin == 'btc') {
      btc.value = event.target.value
      btcUsd.value = (event.target.value * btcPrice.value)?.toFixed(2)
      return
    } 
    if(coin == 'eth') {
      eth.value = event.target.value
      ethUsd.value = (event.target.value * ethPrice.value)?.toFixed(2)
      return
    } 
    if(coin == 'sol') {
      sol.value = event.target.value
      solUsd.value = (event.target.value * solPrice.value)?.toFixed(2)
      return
    } 
  }

  const convertBack = (event, coin) => {
    if(coin == 'btc') {
      btcUsd.value = event.target.value
      btc.value = (event.target.value / btcPrice.value)?.toFixed(6)
    }
    if(coin == 'eth') {
      ethUsd.value = event.target.value
      eth.value = (event.target.value / ethPrice.value)?.toFixed(6)
    }
    if(coin == 'sol') {
      solUsd.value = event.target.value
      sol.value = (event.target.value / solPrice.value)?.toFixed(6)
    }
  }

  const truncatedAddress = computed(() => {
    if (userAddress.value.length > 12) {
      const firstPart = userAddress.value.slice(0, 7); // First 7 characters
      const lastPart = userAddress.value.slice(-5);   // Last 5 characters
      return `${firstPart}...${lastPart}`;
    }
    return userAddress.value;
  });

  // Initialize RPC/injected provider
  const web3 = new Web3();

  // Register the plugin
  web3.registerPlugin(new ChainlinkPlugin());

  async function getBTCPrice() {
    const btcpriceLocal = await web3.chainlink.getPrice(MainnetPriceFeeds.BtcUsd);
    const formattedPrice = (btcpriceLocal.answer / BigInt(1e8)).toString();
    btcPrice.value = parseFloat(formattedPrice).toFixed(2);
  }

  async function getETHPrice() {
    const ethPriceLocal = await web3.chainlink.getPrice(MainnetPriceFeeds.EthUsd);
    const formattedPrice = (ethPriceLocal.answer / BigInt(1e8)).toString();
    ethPrice.value = parseFloat(formattedPrice).toFixed(2);
  }

  async function getSOLPrice() {
    const solPriceLocal = await web3.chainlink.getPrice(MainnetPriceFeeds.SolUsd);
    const formattedPrice = (solPriceLocal.answer / BigInt(1e8)).toString();
    solPrice.value = parseFloat(formattedPrice).toFixed(2);
  }


  const loginWithMetaMask = async() => {
    if (typeof window.ethereum !== 'undefined') {
      if (!loading.value) {
        try {
          loading.value = true; 
          
          await window.ethereum.request({ method: 'eth_requestAccounts' });
          const web3 = new Web3(window.ethereum);

          const accounts = await web3.eth.getAccounts();
          userAddress.value = accounts[0];
          token.value = userAddress.value

          console.log('Connected account:', userAddress.value);
        } catch (error) {
          console.error('User denied account access', error);
        } finally {
          loading.value = false;
        }
      } else {
        console.log('Already processing request, please wait.');
      }
    } else {
      alert('MetaMask is not installed');
    }
  }

  const logout = () => {
    userAddress.value = null;
    token.value = null
    console.log('User logged out');
  }

  onMounted(async() => {
    userAddress.value = token.value;
    await getBTCPrice()
    await getETHPrice()
    await getSOLPrice()
  })

</script>
