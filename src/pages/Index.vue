<template>
  <q-page class="flex flex-center App">
    <div class="container">
      <div class="header-container">
        <p class="header gradient-text">My NFT Collection</p>
        <p class="sub-text">Each unique. Each beautiful. Discover your NFT today.</p>
        <button
          v-if="!current_account"
          @click="connectWallet"
          class="cta-button connect-wallet-button"
        >
          Connect to Wallet
        </button>
        <button
          @click="askContractToMintNft"
          v-else
          class="cta-button connect-wallet-button"
        >
          Mint NFT
        </button>
      </div>
      <code>{{current_mint_count}}/50 minted so far</code>
      <div class="footer-container">
        <img alt="Twitter Logo" class="twitter-logo" src="../assets/twitter-logo.svg" />
        <a class="footer-text" :href="TWITTER_LINK" target="_blank" rel="noreferrer"
          >built on {{ TWITTER_HANDLE }}</a
        >
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { ethers } from "ethers";
import ABI from "../artifacts/contracts/MyNFT.sol/MyNFT.json";
export default defineComponent({
  name: "PageIndex",
  setup() {
    const TWITTER_HANDLE = "johnoba17";
    const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
    const OPENSEA_LINK = "";
    const TOTAL_MINT_COUNT = 50;
    const CONTRACT_ADDRESS = "0xBAa7c673CebC0e76380ec647230E5ABeB70f6011";
    const current_mint_count = ref(null)
    const current_account = ref(null);

    /**
     * Get the current mint Count
     */
    const getCurrentMintCount = async () => {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(
            CONTRACT_ADDRESS,
            ABI.abi,
            signer
          );

          // console.log("Going to pop wallet now to pay gas...");
          current_mint_count.value = await connectedContract.getTotalMintCount();

        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    };
    const checkIfWalletIsConnected = async () => {
      /*
       * First make sure we have access to window.ethereum
       */
      const { ethereum } = window;

      if (!ethereum) {
        console.log("Make sure you have metamask!");
        return;
      } else {
        console.log("We have the ethereum object", ethereum);

      }

      /*
       * Check if we're authorized to access the user's wallet
       */
      const accounts = await ethereum.request({ method: "eth_accounts" });

      /*
       * User can have multiple authorized accounts, we grab the first one if its there!
       */
      if (accounts.length !== 0) {
        const account = accounts[0];

        setCurrentAccount(account);

        setupEventListener();
      } else {
        console.log("No authorized account found");
      }
    };

    const setCurrentAccount = (account) => {
      current_account.value = account;
    };

    // Call functions on mount
    checkIfWalletIsConnected();
    getCurrentMintCount();


    const connectWallet = async () => {
      try {
        const { ethereum } = window;

        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }

        /*
         * Fancy method to request access to account.
         */
        const accounts = await ethereum.request({
          method: "eth_requestAccounts",
        });

        /*
         * Boom! This should print out public address once we authorize Metamask.
         */
        console.log("Connected", accounts[0]);
        setCurrentAccount(accounts[0]);

        setupEventListener();
      } catch (error) {
        console.log(error);
      }
    };

    const askContractToMintNft = async () => {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(
            CONTRACT_ADDRESS,
            ABI.abi,
            signer
          );

          console.log("Going to pop wallet now to pay gas...");
          let nftTxn = await connectedContract.makeAnNFT();

          console.log("Mining...please wait.");
          await nftTxn.wait();

          console.log(
            `Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`
          );
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    };


    const setupEventListener = async () => {
      // Most of this looks the same as our function askContractToMintNft
      try {
        const { ethereum } = window;

        if (ethereum) {
          // Same stuff again
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(
            CONTRACT_ADDRESS,
            ABI.abi,
            signer
          );

          // THIS IS THE MAGIC SAUCE.
          // This will essentially "capture" our event when our contract throws it.
          // If you're familiar with webhooks, it's very similar to that!
          connectedContract.on("NewEpicNFTMinted", (from, tokenId) => {
            console.log(from, tokenId.toNumber());
            current_mint_count.value = tokenId.toNumber()
            console.log(current_mint_count.value)
            alert(
              `Hey there! We've minted your NFT and sent it to your wallet. It may be blank right now. It can take a max of 10 min to show up on OpenSea. Here's the link: https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`
            );
          });

          console.log("Setup event listener!");
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    };
    return {
      TWITTER_HANDLE,
      TWITTER_LINK,
      OPENSEA_LINK,
      TOTAL_MINT_COUNT,
      current_account,
      connectWallet,
      askContractToMintNft,
      current_mint_count
    };
  },
});
</script>
