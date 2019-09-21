<template>
  <v-layout fluid fill-height column justify-center class="pa-6">
    <v-card fill-height>
      <v-card-title class="justify-center">
        <v-img src="../assets/btc.png" contain height="100px" width="100px" />
      </v-card-title>
      <v-card-title class="justify-center">1 BTC = ${{this.prices.USD}}</v-card-title>
      <v-card-text class="text-center">In the next 15 seconds will Bitcoin's price go up or down?</v-card-text>
      <v-card-actions end>
        <v-spacer />
        <v-btn v-on:click="buttonChosen(2)" :disabled="!this.gameStatus==0" color="success">UP</v-btn>
        <v-spacer />
        <v-btn v-on:click="buttonChosen(1)" :disabled="!this.gameStatus==0" color="grey">NO CHANGE</v-btn>
        <v-spacer />
        <v-btn v-on:click="buttonChosen(0)" :disabled="!this.gameStatus==0" color="error">DOWN</v-btn>
        <v-spacer />
      </v-card-actions>
    </v-card>
    <br />
    <v-card>
      <v-card-text class="text-center">
        Prediction locked in at: ${{this.priceAtPrediction}}
        <br />
        Price after 15 seconds: ${{this.priceAfterPrediction}}
        <br />
        <v-chip
          :active="!predictionResultActive"
          :color="this.predictionResult==1 ? `success` : `error`"
        >
        {{this.printPriceChange()}} 
        {{this.predictionResult==1 ? "Prediction correct! +10%" : "Prediction incorrect! -10%"}}
        </v-chip>
      </v-card-text>
    </v-card>
    <br />
    <v-card>
      <v-card-title class="justify-space-around">
        <v-img src="../assets/rocket.png" contain height="75px" width="75px" />Reach the moon to win (100%)
        <v-img src="../assets/moon.png" contain height="75px" width="75px" />
      </v-card-title>
      <v-card-text class="text-center">"Rocketing Finances", get it?</v-card-text>
      <v-card-actions fill-height>
        <v-progress-linear color="white" :value="this.score"></v-progress-linear>
        <!-- YOU WON -->
      </v-card-actions>
      <v-card-text class="text-center">
          <v-btn v-on:click="reset()" :disabled="!isWon" color="success">{{winMessage}}</v-btn>
      </v-card-text>
    </v-card>
  </v-layout>
</template>

<script>
export default {
  //Gives us an an array to hold the bitcoin prices
  data() {
    return {
      priceAtPrediction: 0,
      priceAfterPrediction: 0,
      priceChange: 0,
      score: 0,
      predictionResultActive: true,
      predictionResult: 0, // 0=false, 1=true
      buttonPressed: 0, // 0=down, 1=nochange, 2=up
      gameStatus: 0, // status: 0=choosing, 1=chose, 2=display_result 3=win
      isWon: false,
      winMessage: "...",
      prices: []
    };
  },
  methods: {
    //With this function I wanted it to get the current price and append to the array.
    get: function() {
      this.$http
        .get(
          "https://min-api.cryptocompare.com/data/price?fsym=BTC&tsyms=USD&api_key=%7B9c17742871b6291075518c7a3f62a2cef462f61028c1195e40c13eac0bce1c6b%22"
        )
        .then(data => {
          //console.log(data);
          this.prices = data.body;
        });
    },

    buttonChosen: function(buttonVal) {
      //disable buttons and change gameStatus
      this.gameStatus = 1; // chose
      this.buttonPressed = buttonVal; // 0=down 1=nochange 2=up

      this.priceAtPrediction = this.prices.USD;
      this.priceAfterPrediction = 0;
    },

    printPriceChange: function() {
      if (this.priceChange > 0) {
        return "Price increased: +" + this.priceChange + ".";
      } else if (this.priceChange < 0) {
        return "Price decreased: " + this.priceChange + ".";
      } else {
        return "Price stayed the same: 0.";
      }
    },

    calculateResult: function() {
      this.priceChange = (this.priceAfterPrediction - this.priceAtPrediction).toFixed(2);
      if (this.priceChange > 0) {
        // increase in price
        if (this.buttonPressed == 2) {
          // prediction correct
          this.predictionResult = 1; // win
          this.score += 10;
        } else if (this.buttonPressed == 0 || this.buttonPressed == 1) {
          //prediction incorrect
          this.predictionResult = 0; // lose
          if (this.score > 0) {
            this.score -= 10;
          }
        }
      } else if (this.priceChange < 0) {
        // decrease in price
        if (this.buttonPressed == 0) {
          // prediction correct
          this.predictionResult = 1; // win
          this.score += 10;
        } else if (this.buttonPressed == 1 || this.buttonPressed == 2) {
          //prediction incorrect
          this.predictionResult = 0; // lose
          if (this.score > 0) {
            this.score -= 10;
          }
        }
      } else {
        // no change in price
        if (this.buttonPressed == 1) {
          // prediction correct
          this.predictionResult = 1; // win
          this.score += 10;
        } else if (this.buttonPressed == 0 || this.buttonPressed == 2) {
          //prediction incorrect
          this.predictionResult = 0; // lose
          if (this.score > 0) {
            this.score -= 10;
          }
        }

      }
      this.predictionResultActive = false;
      setTimeout(() => {
        this.gameStatus = 0;
        this.predictionResultActive = true;
        // reset values
        this.priceAfterPrediction = 0;
        this.priceAtPrediction = 0;
      }, 4000);
    },

    reset: function() {
      // reset score, gamestatus, and winMessage
      this.score = 0,
      this.gameStatus = 0,
      this.winMessage = "..."; 
    }
  },
  //When the page, loads it will automatially retrieve the current price of bitocin and set price to it.
  created() {
    this.$http
      .get(
        "https://min-api.cryptocompare.com/data/price?fsym=BTC&tsyms=USD&api_key=%7B9c17742871b6291075518c7a3f62a2cef462f61028c1195e40c13eac0bce1c6b%22"
      )
      .then(data => {
        //console.log(data);
        this.prices = data.body;
        //this.mounted()
      });
  },
  mounted() {
    //This function waits ten seconds and then calls the get method
    window.setInterval(() => {
      this.get();
    }, 10000);
  },
  watch: {
    prices: function(val) {
      if (this.gameStatus == 1) {
        this.priceAfterPrediction = val.USD;
      }
    },
    priceAfterPrediction: {
      handler: function(val) {
        if (val != 0) {
          // once the price updates, then calculate result
          this.calculateResult();
        }
      }
    },
    score: function(val) {
      if (val >= 100) {
        this.gameStatus = 3; // WIN
        this.isWon = true;
        this.winMessage = "YOU WIN! PLAY AGAIN?"
      }
    }
  }
};
</script>