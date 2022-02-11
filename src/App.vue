<template>
  <div class="main">
    <div class="header pt-4">SOLANA PERFECT NFT</div>
    <div class="d-flex justify-content-center align-items-center">
      <img  class="logo" src="./assets/collect.jpg" alt="">
      <p class="subheader mt-3">Mint your personal NFT collection now!</p>
    </div>

    <div>

      <button v-if="!walletAddress"  class="wallet-btn mt-3" @click="connectToWallet">{{ !walletConnected ? 'Connect to wallet' : 'Disconnect' }}</button>
      <button v-else class="wallet-btn mt-3" @click="disconnectWallet">{{ !walletConnected ? 'Connect to wallet' : 'Disconnect'  }}</button>
<!--      <img class="d-block mx-auto mt-5 waiting-box" :class="showWaiting ? 'show' : ''" src="./assets/waiting.gif" alt="">-->
    </div>
  </div>

  <div class="footer">
    <img src="https://gateway.pinata.cloud/ipfs/QmVYRRvrJRNNfhxnp12dW4zYMS5iXwavxCk8oRUPwhdDQk" alt="">
    <a href="https://twitter.com/amer1canWM" target="_blank" rel="noreferrer">Seeing on Twitter</a>
  </div>
</template>

<script>

// initialized config for a candy machine with publickey: 6NNsjaZpEkdq3pD27WobmPA13FMZA8NBdVC9vz6jujxD

import { Connection } from '@solana/web3.js';
import { Program, Provider, web3 } from '@project-serum/anchor';

const candyMachineProgram = new web3.PublicKey(
    '6NNsjaZpEkdq3pD27WobmPA13FMZA8NBdVC9vz6jujxD'
);
const opts = {
  preflightCommitment: 'processed',
}


export default {
  name: 'App',
  data() {
    return {
      walletConnected: null,
      walletAddress: null,
    }
  },
  async mounted() {
    await this.checkIfWalletConnect()
    await this.getCandyMachineState()
  },
  computed: {},
  methods: {
    async checkIfWalletConnect() {
      try {
        const {solana} = window;
        if (solana) {
          if (solana.isPhantom) {
            console.log('Phantom wallet found!', solana)
            const response = await solana.connect({onlyIfTrusted: true})
            console.log('Trusted Connection with Public Key:', response.publicKey.toString())
            this.walletAddress = response.publicKey.toString()
            this.walletConnected = true
          }
        } else {
          alert('Solana object not found! Get a Phantom wallet!')
        }

      } catch (err) {
        console.log('CheckWalletConnection Error!', err)
      }
    },
    async connectToWallet() {
      const {solana} = window;
      if (solana) {
        const response = await solana.connect()
        console.log('Connected with Public Key:', response.publicKey.toString())
        this.walletAddress = response.publicKey.toString()
        this.walletConnected = true
      }
    },
    disconnectWallet() {
      window.solana.disconnect()
      console.log('Disconnected')
      this.walletAddress = null
      this.walletConnected = false
    },
    getProvider() {
      const rpcHost = 'https://explorer-api.devnet.solana.com';
      const connection = new Connection(rpcHost);
      const provider = new Provider(
          connection,
          window.solana,
          opts.preflightCommitment
      )
      return provider;
    },
    async getCandyMachineState() {
      const provider = this.getProvider();
      const idl = await Program.fetchIdl(candyMachineProgram, provider);
      console.log('idl',idl)
      const program = new Program(idl, candyMachineProgram, provider);
      const candyMachine = await program.account.candyMachine.fetch(
          process.env.VUE_APP_CANDY_MACHINE_ID
      );

      const itemsAvailable = candyMachine.data.itemsAvailable.toNumber();
      const itemsRedeemed = candyMachine.itemsRedeemed.toNumber();
      const itemsRemaining = itemsAvailable - itemsRedeemed;
      const goLiveData = candyMachine.data.goLiveData.toNumber();
      const presale = candyMachine.data.whitelistMintSettings &&
          candyMachine.data.whitelistMintSettings.presale &&
          (!candyMachine.data.goLiveData ||
          candyMachine.data.goLiveData.toNumber() > new Date().getTime() / 1000);
      const goLiveDateTimeString = `${new Date(goLiveData * 1000).toGMTString()}`
      console.log({
        itemsAvailable,
        itemsRedeemed,
        itemsRemaining,
        goLiveData,
        goLiveDateTimeString,
        presale,
      });

    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background: black;
  display: flex;
  flex-direction: column;
  height: 100vh;
}
.main {
  flex-grow: 1;
}
.header {
  font-size: 50px;
  font-weight: bold;
  background: -webkit-linear-gradient(left, #fcd643 30%, #2d8003 60%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
.logo {
  margin-right: 30px;
  height: 70px;
  border-radius: 20px;
}
.subheader {
  font-size: 30px;
  color: white;
}
.wallet-btn {
  font-size: 22px;
  padding: 0.5em;
  border-radius: 15px;
  background: -webkit-linear-gradient(left, #fcd643 30%, #2d8003 60%);
  background-size: 200% 200%;
  animation: gradient-animation 4s ease infinite;
  align-self: center;
}
.waiting-box {
  width: 150px;
  height: 150px;
  display: none;
}
.waiting-box.show {
  display: block;
}

.footer {
  padding: 20px 0px;
}
.footer a {
  color: coral;
  font-size: 18px;
  font-weight: bold;
  text-decoration: none;
  transition: all 0.2s ease-in;
}
.footer a:hover {
  color: #ff4444;
  transition: all 0.2s ease-in;
}
.footer img {
  width: 30px;
}

/*------------------------------------------------------*/
/* TOAST */

@keyframes fadeout {
  from {
    top: 60px;
    opacity: 1;
  }
  to {
    top: 0px;
    opacity: 0;
  }
}


/* SHAKE */
.shake-slow {
  display: inherit;
  transform-origin: center center;
  animation-play-state: running
}
@keyframes shake-slow {
  2% {
    transform: translate(2px, 1px) rotate(-0.5deg); }
  4% {
    transform: translate(-5px, -9px) rotate(-2.5deg); }
  6% {
    transform: translate(5px, 3px) rotate(2.5deg); }
  8% {
    transform: translate(5px, 4px) rotate(1.5deg); }
  10% {
    transform: translate(-4px, -8px) rotate(-0.5deg); }
  12% {
    transform: translate(-2px, -9px) rotate(1.5deg); }
  14% {
    transform: translate(0px, 6px) rotate(3.5deg); }
  16% {
    transform: translate(8px, 2px) rotate(2.5deg); }
  18% {
    transform: translate(-4px, -1px) rotate(1.5deg); }
  20% {
    transform: translate(10px, 2px) rotate(-2.5deg); }
  22% {
    transform: translate(-2px, -4px) rotate(1.5deg); }
  24% {
    transform: translate(-8px, 5px) rotate(-0.5deg); }
  26% {
    transform: translate(-3px, -5px) rotate(3.5deg); }
  28% {
    transform: translate(6px, 2px) rotate(2.5deg); }
  30% {
    transform: translate(-5px, -2px) rotate(2.5deg); }
  32% {
    transform: translate(5px, -6px) rotate(-2.5deg); }
  34% {
    transform: translate(5px, -7px) rotate(-1.5deg); }
  36% {
    transform: translate(3px, -9px) rotate(0.5deg); }
  38% {
    transform: translate(6px, -2px) rotate(-0.5deg); }
  40% {
    transform: translate(-4px, -2px) rotate(0.5deg); }
  42% {
    transform: translate(7px, -8px) rotate(1.5deg); }
  44% {
    transform: translate(-4px, 10px) rotate(-1.5deg); }
  46% {
    transform: translate(-3px, 8px) rotate(-1.5deg); }
  48% {
    transform: translate(3px, 6px) rotate(-0.5deg); }
  50% {
    transform: translate(4px, -1px) rotate(-2.5deg); }
  52% {
    transform: translate(1px, 5px) rotate(-0.5deg); }
  54% {
    transform: translate(-3px, 7px) rotate(-2.5deg); }
  56% {
    transform: translate(3px, 3px) rotate(-2.5deg); }
  58% {
    transform: translate(-8px, -5px) rotate(-1.5deg); }
  60% {
    transform: translate(1px, -9px) rotate(3.5deg); }
  62% {
    transform: translate(0px, -3px) rotate(-0.5deg); }
  64% {
    transform: translate(4px, 2px) rotate(2.5deg); }
  66% {
    transform: translate(4px, 10px) rotate(1.5deg); }
  68% {
    transform: translate(1px, 2px) rotate(-2.5deg); }
  70% {
    transform: translate(-9px, 10px) rotate(0.5deg); }
  72% {
    transform: translate(-5px, -8px) rotate(0.5deg); }
  74% {
    transform: translate(-3px, -8px) rotate(3.5deg); }
  76% {
    transform: translate(-1px, 8px) rotate(3.5deg); }
  78% {
    transform: translate(-5px, -4px) rotate(0.5deg); }
  80% {
    transform: translate(-6px, 1px) rotate(1.5deg); }
  82% {
    transform: translate(-5px, -7px) rotate(1.5deg); }
  84% {
    transform: translate(1px, -6px) rotate(-0.5deg); }
  86% {
    transform: translate(-8px, -4px) rotate(3.5deg); }
  88% {
    transform: translate(-6px, 7px) rotate(-1.5deg); }
  90% {
    transform: translate(9px, 6px) rotate(-2.5deg); }
  92% {
    transform: translate(9px, -8px) rotate(3.5deg); }
  94% {
    transform: translate(3px, 3px) rotate(2.5deg); }
  96% {
    transform: translate(-6px, -7px) rotate(2.5deg); }
  98% {
    transform: translate(5px, -2px) rotate(-0.5deg); }
  0%, 100% {
    transform: translate(0, 0) rotate(0); } }
.shake-slow {
  animation-name: shake-slow;
  animation-duration: 5s;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}
/*------------------------------------------------------*/
.shake-little {
  display: inherit;
  transform-origin: center center;
  animation-play-state: running;
}

@keyframes shake-little {
  2% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  4% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  6% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  8% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  10% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  12% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  14% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  16% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  18% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  20% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  22% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  24% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  26% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  28% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  30% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  32% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  34% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  36% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  38% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  40% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  42% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  44% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  46% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  48% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  50% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  52% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  54% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  56% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  58% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  60% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  62% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  64% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  66% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  68% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  70% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  72% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  74% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  76% {
    transform: translate(0px, 1px) rotate(0.5deg); }
  78% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  80% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  82% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  84% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  86% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  88% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  90% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  92% {
    transform: translate(1px, 1px) rotate(0.5deg); }
  94% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  96% {
    transform: translate(1px, 0px) rotate(0.5deg); }
  98% {
    transform: translate(0px, 0px) rotate(0.5deg); }
  0%, 100% {
    transform: translate(0, 0) rotate(0); } }

.shake-little {
  animation-name: shake-little;
  animation-duration: 100ms;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}

/* KeyFrames */
@-webkit-keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
@-moz-keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
@keyframes gradient-animation {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
</style>
