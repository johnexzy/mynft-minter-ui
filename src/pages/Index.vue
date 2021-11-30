<template>
  <q-page class="flex flex-center App">
    <div class="container">
      <div class="header-container">
        <p class="header gradient-text">Mint your Art to NFT</p>
        <!-- <p class="sub-text">
          Each unique. Each beautiful. Discover your NFT today.
        </p> -->
        <q-form @submit="onSubmit" class="q-gutter-md q-py-lg q-my-lg">
          <div v-if="current_account" class="row justify-center">
            <div class="col-12 col-md-5 col-lg-5">
              <div class="row q-col-gutter-sm">
                <div class="col-12 col-md-10 col-lg-10">
                  <div class="text-subtitle1 text-primary text-left">
                    NFT Name
                  </div>
                  <div class="text-caption text-grey-7  text-left q-mb-md">
                    Provide a name for this NFT
                  </div>
                  <q-input
                    class="custom-form-input"
                    v-model="IPFS_JSON.name"
                    ref="name"
                    dense
                    filled
                    placeholder="Name or title"
                    hint
                    lazy-rules
                    :rules="[$rules.required('Please enter name for nft')]"
                  >
                  </q-input>
                </div>
              </div>
              <div class="row q-col-gutter-sm">
                <div class="col-12 col-md-12 col-lg-12">
                  <div class="text-subtitle1 text-primary text-left">
                    NFT Description
                  </div>
                  <div class="text-caption text-left text-grey-7 q-mb-md">
                    Share more information about this nft
                  </div>
                  <q-input
                    class="custom-form-input"
                    v-model="IPFS_JSON.description"
                    ref="description"
                    type="textarea"
                    filled
                    dense
                    hint
                    lazy-rules
                    :rules="[
                      $rules.required('Description is required'),
                      $rules.maxLength(
                        200,
                        'Description should not be larger than 200 chars'
                      ),
                    ]"
                  >
                  </q-input>
                </div>
              </div>
              <div class="row q-col-gutter-sm">
                <div class="col-12 col-md-12 col-lg-12">
                  <div class="text-subtitle1 text-primary text-left">
                    Select Image
                  </div>
                  <div class="text-caption text-grey-7 text-left q-mb-md">
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
                <q-btn
                  v-if="!current_account"
                  @click="connectWallet"
                  no-wrap
                  flat
                  class="cta-button connect-wallet-button"
                >
                  Connect to Wallet
                </q-btn>
                <q-btn
                  type="submit"
                  no-wrap
                  flat
                  v-else-if="current_account && !minting"
                  class="cta-button connect-wallet-button"
                >
                  {{ labelBtn }}
                </q-btn>
                <q-btn
                  v-show="minting"
                  no-wrap
                  flat
                  :loading="minting"
                  class="cta-button connect-wallet-button"
                >
                  Minting...
                </q-btn>
              </div>
            </div>
          </div>
          <q-btn
            v-else
            @click="connectWallet"
            no-wrap
            flat
            class="cta-button connect-wallet-button"
          >
            Connect to Wallet
          </q-btn>
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
    const CONTRACT_ADDRESS = "0x045388AD8078615958F784455F5C71cEc049b48B";
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

    // getCurrentMintCount();

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
    const IPFS_JSON = ref({
      description: "",
      image: "ipfs://YOUR_ASSET_CID",
      name: "",
    });
    const UploadJson = async () => {
      try {
        const IPFS = {
          name: IPFS_JSON.value.name,
          image: IPFS_JSON.value.image,
          description: IPFS_JSON.value.description,
        };
        await axios
          .post(
            "pinning/pinJSONToIPFS",
            IPFS,
            {
              headers: {
                "Content-Type": "application/json",
              },
            }
          )
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
      // const pinataMetadata = JSON.stringify({
      //   name: "Oba John Test",
      //   keyvalues: {
      //     company: "John Inc",
      //   },
      // });
      // formData.append("pinataMetadata", pinataMetadata);
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
            IPFS_JSON.value.image = `ipfs://${res.data.IpfsHash}`;

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
      IPFS_JSON,
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
