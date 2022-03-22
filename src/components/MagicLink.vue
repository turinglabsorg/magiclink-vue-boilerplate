<template>
  <div class="hello">
    <div class="container">
      <h1 style="margin-bottom: 30px">Vue / MagicLink</h1>
      <hr />
      <div v-if="!checking">
        <div v-if="!isLoggedIn">
          <p>Please sign up or login</p>
          <input
            type="email"
            name="email"
            v-model="email"
            placeholder="Enter your email"
          />
          <button v-on:click="loginWithEmail">Send</button><br />
        </div>
        <div v-if="isLoggedIn">
          <b>User address is:</b><br />
          {{ userAddress }}<br /><br />
          <b>Balance is:</b><br />
          {{ userBalance }} ETH<br /><br />
          <b>Network is:</b><br />
          {{ network.name }}
          <br /><br />
          <hr />
          <br />
          You're minting
          <a
            href="https://testnets.opensea.io/collection/nonft-v2"
            target="_blank"
            >"nothing"</a
          ><br />for 0.01 ETH at
          <b>0xf81046bafce92bdac543f2a91fa91c16ba40abde</b><br /><br />
          <div v-if="isSending" style="margin-bottom:30px">Sending transaction, please wait..</div>
          <button
            style="margin-bottom: 30px"
            v-if="!tx && !isSending"
            v-bind:disabled="userBalance <= 0.01"
            v-on:click="mint"
          >
            MINT
          </button>
          <div v-if="tx">
            <b>Tx submitted at:</b><br />
            <a :href="'https://rinkeby.etherscan.io/tx/' + tx.hash">{{
              tx.hash
            }}</a
            ><br /><br />
            <div v-if="receipt"><b>Receipt is:</b><br />{{ receipt }}</div>
          </div>
          <hr />
          <br /><br />
          <button v-on:click="logout">Logout</button>
        </div>
      </div>
      <div v-if="checking">Checking Magic status..</div>
    </div>
  </div>
</template>

<script>
import { Magic } from "magic-sdk";
import { ethers } from "ethers";

export default {
  name: "MagicLink",
  data() {
    return {
      magic: {},
      email: "",
      userMetadata: {},
      userAddress: "",
      userBalance: 0,
      checking: true,
      isLoggedIn: false,
      isSending: false,
      network: {},
      // This must be changed
      contract_address: process.env.VUE_APP_CONTRACT_ADDRESS,
      ABI: [
        {
          inputs: [],
          name: "buyNFT",
          outputs: [],
          stateMutability: "payable",
          type: "function",
          payable: true,
        },
      ],
      tx: "",
      receipt: "",
    };
  },
  mounted() {
    const app = this;
    app.magic = new Magic(process.env.VUE_APP_MAGIC_PUB_KEY, {
      network: process.env.VUE_APP_NETWORK,
    });
    app.render();
  },
  methods: {
    async loginWithEmail() {
      const app = this;
      if (app.email.length > 0) {
        await app.magic.auth.loginWithMagicLink({ email: app.email });
        app.render();
      }
    },
    async render() {
      const app = this;
      const isLoggedIn = await app.magic.user.isLoggedIn();
      console.log("User is logged?", isLoggedIn);
      if (isLoggedIn) {
        app.isLoggedIn = true;
        const provider = new ethers.providers.Web3Provider(app.magic.rpcProvider);
        console.log("User is logged in..");
        app.userMetadata = await app.magic.user.getMetadata();
        const signer = provider.getSigner();
        app.network = await provider.getNetwork();
        console.log("Network is: ", app.network);
        app.userAddress = await signer.getAddress();
        console.log("Address is " + app.userAddress);
        app.userBalance = ethers.utils.formatEther(
          await provider.getBalance(app.userAddress)
        );
        console.log("User balance is " + app.userBalance);
        app.checking = false;
      } else {
        app.checking = false;
      }
    },
    async mint() {
      const app = this;
      const isLoggedIn = await app.magic.user.isLoggedIn();
      console.log("User is logged?", isLoggedIn);
      if (isLoggedIn) {
        app.isSending = true;
        const provider = new ethers.providers.Web3Provider(app.magic.rpcProvider);
        console.log("User is logged in..");
        try {
          app.userMetadata = await app.magic.user.getMetadata();
          const signer = provider.getSigner();
          let contract = new ethers.Contract(
            app.contract_address,
            app.ABI,
            signer
          );
          app.tx = await contract.buyNFT({
            value: ethers.utils.parseEther("0.01"),
          });
          console.log("Submitted:", app.tx);
          app.receipt = await app.tx.wait();
          console.log("Completed:", app.receipt);
          app.isSending = false;
        } catch (e) {
          console.log(e);
          app.isSending = false;
        }
      }
    },
    async logout() {
      const app = this;
      await app.magic.user.logout();
      app.isLoggedIn = false;
    },
  },
};
</script>