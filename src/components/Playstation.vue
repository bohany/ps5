<template>
  <div class="titleCard">
    <h1>{{ testMessage }}</h1>
    <button class="btn" @click="showHide">Show/Hide Instructions</button>
    <p v-if="toggleInstructions">
      Click the "CHECK ALL STORES" button to start the program and check if the
      PS5 is available at all stores listed below. The stores will then
      auto-refresh every ten minutes. If in stock, this app will play a song to alert you, at that time you can click the "GRAB IT" button on the
      individual store to buy it there. Refresh the page or close the browser 
      tab to stop auto-checking.
    </p>
    <button class="btn" @click="start">CHECK ALL STORES</button>
    <p>Stores will refresh in {{counter}} seconds</p>
  </div>

  <div class="container">
    <div class="card">
      <img
        src="https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.seeklogo.net%2Fwp-content%2Fuploads%2F2013%2F06%2Ftarget-australia-vector-logo.png&f=1&nofb=1"
      />
      <p v-if="targetAvailability == ''">
        Click "Check All Stores" above to Check Availability
      </p>
      <p v-else-if="targetAvailability == 'PRE_ORDER_UNSELLABLE'">
        Still not in stock at Target
      </p>
      <p v-else>
        It's here! Get it! There are {{ targetStock }} units in stock!
      </p>
      <a
        href="https://www.target.com/p/playstation-5-console/-/A-81114595?crushProductId=81114595"
        target="_blank"
        ><button class="btn">GRAB IT!</button></a
      >
      <p style="display: none">{{ targetResult }}</p>
    </div>

    <div class="card">
      <img
        src="https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.evolveyourweddingbusiness.com%2Fwp-content%2Fuploads%2F2017%2F03%2FPNGPIX-COM-Walmart-Vertical-Logo-PNG-Transparent.png&f=1&nofb=1"
      />
      <p v-if="walmartAvailability == null">
        Click "Check All Stores" above to Check Availability
      </p>
      <p v-else-if="walmartAvailability == false">
        Still not in stock at Walmart
      </p>
      <p v-else>
        It's here! Get it now!
      </p>
      <a
        href="https://www.walmart.com/ip/PlayStation5-Console/363472942"
        target="_blank"
        ><button class="btn">GRAB IT!</button></a
      >
      <p style="display: none">{{ walmartResult }}</p>
    </div>
  </div>

  <div class='container'>
    <div class="card">
      <img
        src="https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fstovetecstore.net%2Fwp-content%2Fuploads%2F2016%2F09%2Famazon-transparent-logo-a.png&f=1&nofb=1"
      />
      <p v-if="amazonAvailability === null">
        Click "Check All Stores" above to Check Availability
      </p>
      <p v-else-if="amazonAvailability == true">Still not in stock at Amazon</p>
      <p v-else>It's here! Get it now!</p>
      <a href="https://www.amazon.com/dp/B08FC5L3RG/?psc=1" target="_blank"
        ><button class="btn">GRAB IT!</button></a
      >
      <p style="display: none">{{ amazonResult }}</p>
    </div>

    <div class="card">
      <img
        src="https://logos-download.com/wp-content/uploads/2016/02/Best_Buy_Logo_Transparent.png"
      />
      <p>{{ bbResult ? "IN STOCK NOW!" : "Still not available" }}</p>
      <a href="https://api.bestbuy.com/click/-/6426149/cart" target="_blank"
        ><button class="btn">GRAB IT!</button></a
      >
    </div>
  </div>
  

  <audio
    id="song"
    src="https://upload.wikimedia.org/wikipedia/commons/2/2a/La_Traviata_-_Act_1_-_Libiamo_ne%27_lieti_calici.ogg"
  />
  <button @click="playSong">Sound Check</button>
</template>
<script>
import Axios from "axios";

export default {
  name: "Playstation",
  data() {
    return {
      testMessage: "GET OUR PS5!",
      toggleInstructions: false,
      bbResult: null,
      bbAvailability: "",
      targetResult: null,
      targetAvailability: "",
      targetStock: 0,
      amazonResult: null,
      amazonAvailability: null,
      walmartAvailability: null,
      counter: 600,
    };
  },
  computed: {},
  watch: {
    amazonAvailability() {
      const song = document.querySelector("#song");
      song.loop = true;
      if (this.amazonAvailability == false) {
        alert("It's available at Amazon!");
        song.play();
      }
    },
    targetAvailability() {
      const song = document.querySelector("#song");
      song.loop = true;
      if (this.targetAvailability !== "PRE_ORDER_UNSELLABLE" && "") {
        alert("It's available at Target!");
        song.play();
      }
    },
    walmartAvailability() {
      const song = document.querySelector("#song");
      song.loop = true;
      if (this.walmartAvailability == true) {
        alert("It's available at Walmart!");
        song.play();
      }
    },
    counter(){
      if (this.counter <= 0) {
        this.counter = 600;
      }
    }
  },
  methods: {
    showHide() {
      this.toggleInstructions = !this.toggleInstructions;
    },
    playSong() {
      const song = document.querySelector("#song");
      song.loop = true;
      song.play();
    },
    start() {
      setInterval(()=>{this.counter--}, 1000);
      let checkBB = () => {
        Axios.request({
          baseURL: "https://api.bestbuy.com/v1/products/",
          method: "get",
          params: {
            apiKey: "?apiKey=HprALUgAxjiAaSYuAi8q2LwA",
            sortOptions: "&sort=onlineAvailability.asc",
            showOptions:
              "&show=addToCartUrl,onlineAvailability,onlineAvailabilityText",
            responseFormat: "&format=json",
          },
        }).then((response) => {
          this.bbResult = response.data.products;
          this.bbAvailability = response.data.products.onlineAvailability;
        });
      };

      let checkTarget = () => {
        Axios.request({
          method: "GET",
          url: "https://target1.p.rapidapi.com/products/get-details",
          headers: {
            "content-type": "application/octet-stream",
            "x-rapidapi-host": "target1.p.rapidapi.com",
            "x-rapidapi-key":
              "f76ab48dd8mshc870381c400304dp199e58jsn2a55c36faffa",
            useQueryString: true,
          },
          params: {
            tcin: "81114595",
            store_id: "2776",
          },
        })
          .then((response) => {
            this.targetResult =
              response.data.data.product.available_to_promise.network;
            this.targetAvailability =
              response.data.data.product.available_to_promise.network.availability_status;
            this.targetStock =
              response.data.data.product.available_to_promise.network.available_to_promise_quantity;
          })
          .catch((error) => {
            console.log(error);
          });
      };

      let checkAmazon = () => {
        Axios.request({
          method: "GET",
          url: "https://amazon-products1.p.rapidapi.com/product",
          headers: {
            "content-type": "application/octet-stream",
            "x-rapidapi-host": "amazon-products1.p.rapidapi.com",
            "x-rapidapi-key":
              "f76ab48dd8mshc870381c400304dp199e58jsn2a55c36faffa",
            useQueryString: true,
          },
          params: {
            country: "US",
            asin: "B08FC5L3RG",
          },
        })
          .then((response) => {
            this.amazonResult = response.data;
            this.amazonAvailability = response.data.out_of_stock;
          })
          .catch((error) => {
            console.log(error);
          });
      };

      let checkWalmart = () => {
          Axios.request({
            method:"GET",
            url:"https://axesso-walmart-data-service.p.rapidapi.com/wlm/walmart-lookup-product",
            headers:{
            "content-type":"application/octet-stream",
            "x-rapidapi-host":"axesso-walmart-data-service.p.rapidapi.com",
            "x-rapidapi-key":"f76ab48dd8mshc870381c400304dp199e58jsn2a55c36faffa",
            "useQueryString":true
            },
            params:{
            url:"https://www.walmart.com/ip/PlayStation5-Console/363472942"
            }
            })
            .then(response=>{
              this.walmartAvailability = response.data.available;
              this.walmartResult = response.data;
            })
            .catch((error)=>{
              console.log(error)
            })
      }

      checkBB();
      checkTarget();
      checkAmazon();
      checkWalmart();
      setInterval(checkTarget, 600000);
      setInterval(checkAmazon, 600000);
      setInterval(checkBB, 600000);
      setInterval(checkWalmart, 600000)
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  margin: 1vmin;
}

.titleCard {
  background-color: rgba(233, 210, 148, 0.966);
  border-radius: 10px;
  padding: 4vmin;
  margin-top: -50px;
  box-shadow: 2px 2px 2px grey;
  align-items: center;
  justify-content: center;
  display: flex;
  flex-direction: column;
}

.titleCard:hover {
  box-shadow: 5px 5px 5px grey;
}

p {
  word-wrap: break-word;
}
img {
  border: 1px solid rgba(197, 172, 193, 0.733);
  border-radius: 10%;
  padding: 5px;
  width: 15vw;
  box-shadow: 2px 2px 2px lightslategrey;
  background-color: white;
}
.card {
  margin: 2vmin;
  border: solid 1px darkslategrey;
  box-shadow: 2px 2px 2px grey;
  padding: 3vmin;
  background-color: linen;
  border-radius: 20px;
  width: 30vw;
}

.card:hover {
  margin: 5vmin;
  border: solid 1px darkslategrey;
  box-shadow: 5px 5px 9px grey;
  padding: 5vmin;
  background-color: linen;
  border-radius: 20px;
}

.btn {
  box-shadow: 2px 2px 2px grey;
  background-color: rgb(72, 99, 219);
  border-radius: 5px;
  color: white;
  height: 11vmin;
  width: 24vmin;
  padding: 5px;
  word-wrap: break-word;
  font-size: 16px;
  margin: 2vmin;
}

.btn:hover {
  box-shadow: 5px 5px 5px grey;
  background-color: rgb(72, 99, 219);
  border: solid 1px white;
}
</style>
