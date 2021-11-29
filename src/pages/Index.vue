<template>
  <q-page class="flex flex-center App">
    <div class="container">
      <div class="header-container">
        <p class="header gradient-text">Mint your Art to NFT</p>
        <!-- <p class="sub-text">
          Each unique. Each beautiful. Discover your NFT today.
        </p> -->
        <q-form @submit="onSubmit" class="q-gutter-md q-py-lg q-my-lg">
          <div class="row justify-center">
            <div class="col-12 col-md-5 col-lg-5">
              <div class="row q-col-gutter-sm justify-center">
                <div class="col-12 col-md-12 col-lg-12">
                  <div class="text-subtitle1 text-primary">Select Image</div>
                  <div class="text-caption text-grey-7 q-mb-md">
                    PNG, JPG, or GIF up to 1MB
                  </div>
                  <!-- <q-field ref="image" class="justify-center" dense borderless hint> -->
                    <q-file
                      ref="files"
                      style="display: none"
                      v-model="file"
                      name="file"
                      accept="image/png, image/jpeg, image/gif"
                      max-file-size="1000000"
                    />
                    <q-btn
                      v-if="!file"
                      style="width: 150px !important; height: 150px"
                      @click="$refs.files.pickFiles()"
                      text-color="grey-9"
                      class="justify-center "
                      color="grey-3"
                      unelevated
                      icon="add"
                    />
                    <q-chip
                      v-else
                      class=""
                      icon="ion-image"
                      removable
                      @remove="clearSelectedImage"
                      icon-remove="delete"
                      :label="file.name"
                    />
                  <!-- </q-field> -->
                </div>
              </div>
              <div class="row q-col-gutter-sm q-my-lg justify-center">
                <button
                  v-if="!current_account"
                  @click="connectWallet"
                  class="cta-button connect-wallet-button"
                >
                  Connect to Wallet
                </button>
                <button
                  type="submit"
                  v-else-if="current_account && !minting"
                  class="cta-button connect-wallet-button"
                >
                  {{ labelBtn }}
                </button>
                <button
                  v-show="minting"
                  class="cta-button connect-wallet-button"
                >
                  Minting...
                </button>
              </div>
            </div>
          </div>
        </q-form>
      </div>
      <div class="code">{{ current_mint_count }}/50 minted so far</div>

      <div></div>
      <div class="text-indigo-1" v-if="msg">
        {{ msg }}
      </div>
      <div class="footer-container">
        <img
          alt="Twitter Logo"
          class="twitter-logo"
          src="../assets/twitter-logo.svg"
        />
        <a
          class="footer-text"
          :href="TWITTER_LINK"
          target="_blank"
          rel="noreferrer"
          >built by @{{ TWITTER_HANDLE }}</a
        >
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { ethers } from "ethers";
import axios from "axios";
import ABI from "../artifacts/contracts/MyNFT.sol/MyNFT.json";
export default defineComponent({
  name: "PageIndex",
  setup() {
    const TWITTER_HANDLE = "johnoba17";
    const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
    const TOTAL_MINT_COUNT = 50;
    const CONTRACT_ADDRESS = "0x5205AB26d7a629F2ffb677C463f8929aBA3349b2";
    const current_mint_count = ref(0);
    const current_account = ref(null);
    const minting = ref(false);
    const msg = ref("");
    const labelBtn = ref("Mint NFT");
    // const $api = axios;
    const IPFS_CID = ref(null);
    const checkChainId = async () => {
      let chainId = await ethereum.request({ method: "eth_chainId" });
      console.log("Connected to chain " + chainId);

      // String, hex code of the chainId of the Rinkebey test network
      const rinkebyChainId = "0x4";
      if (chainId !== rinkebyChainId) {
        alert("Please switch to Rinkeby Test Network to Mint NFT!");
      }
    };

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
          const count = await connectedContract.getTotalMintCount();
          current_mint_count.value = parseInt(count);
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    };

    getCurrentMintCount();

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
        checkChainId();
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

        getCurrentMintCount();
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
        getCurrentMintCount();
        setupEventListener();
      } catch (error) {
        console.log(error);
      }
    };

    /**
     * Minting function
     */
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
          let nftTxn = await connectedContract.mintNFT(IPFS_CID.value);
          minting.value = true;
          labelBtn.value = "Mint NFT";
          console.log("Mining...please wait.");
          await nftTxn.wait();
          getCurrentMintCount();
          minting.value = false;
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

    /**
     * Events is emitted when transaction is completed
     */
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
            minting.value = false;
            current_mint_count.value = parseInt(tokenId.toNumber()) + 1;
            // console.log(current_mint_count.value)
            msg.value = `${from} minted NFT number: ${tokenId}. view on https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`;
          });

          console.log("Setup event listener!");
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
      }
    };

    /**
     * Post Images to IPFS
     */

    const IPFSdata = ref(null);
    const IPFS_JSON = {
      description: "Description of your NFT",
      image: "ipfs://YOUR_ASSET_CID",
      name: "A name for your NFT",
    };
    const UploadJson = async () => {
      try {
        await axios
          .post("pinning/pinJSONToIPFS", IPFS_JSON, {
            headers: {
              "Content-Type": "application/json",
            },
          })
          .then((res) => {
            console.log(IPFS_JSON);
            console.log("hash", res.data);
            // IPFS_JSON.value.image = `ipfs://${res.data.IpfsHash}`;
            IPFS_CID.value = `ipfs://${res.data.IpfsHash}`;
            askContractToMintNft();
          });
      } catch (error) {
        console.log("error", error.responds.data);
      }
    };

    const onSubmit = async (evt) => {
      const formData = new FormData(evt.target);
      // const data = [];
      const pinataOptions = JSON.stringify({
        cidVersion: 0,
      });
      const pinataMetadata = JSON.stringify({
        name: "Oba John Test",
        keyvalues: {
          company: "John Inc",
        },
      });
      formData.append("pinataMetadata", pinataMetadata);
      formData.append("pinataOptions", pinataOptions);
      try {
        labelBtn.value = "Uploading...";
        await axios
          .post("pinning/pinFileToIPFS", formData, {
            // maxBodyLength: "Infinity", //this is needed to prevent axios from erroring out with large files
            headers: {
              "Content-Type": `multipart/form-data;`,
            },
          })
          .then((res) => {
            IPFSdata.value = res.data;
            IPFS_JSON.image = `ipfs://${res.data.IpfsHash}`;

            UploadJson();
          });
      } catch (error) {
        console.log("error", error.responds.data);
      }
    };
    const file = ref(null);
    return {
      TWITTER_HANDLE,
      TWITTER_LINK,
      TOTAL_MINT_COUNT,
      current_account,
      minting,
      connectWallet,
      askContractToMintNft,
      current_mint_count,
      file,
      clearSelectedImage() {
        file.value = null;
      },
      onSubmit,
      msg,
      labelBtn,
    };
  },
});
</script>
