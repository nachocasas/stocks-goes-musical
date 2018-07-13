<template>
  <div id="app" class="container">
    <Search v-on:search="search" :firstUse="data==null" />
    <div v-if="loading==true" class="lds-dual-ring"></div>
    <MusicalStock
      v-if="dates != null"
      v-bind:symbol="symbol"
      v-bind:dates="dates"
      v-bind:prices="prices"
      v-bind:minValue="minValue"
      v-bind:maxValue="maxValue"
      v-bind:chartData="collection"
      />
  </div>
</template>

<script>
/* eslint-disable */
import Search from './components/Search'
import MusicalStock from './components/MusicalStock'

import Axios from 'axios';
import _ from 'lodash'

var API_KEY = '8Q0D7JFV40RJL5CX';
var API_URL = `https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&interval=1min&apikey=${API_KEY}`;

export default {
  name: 'App',
  components: {
    Search,
    MusicalStock
  },
  data(){
    return {
      loading: false,
      data: null,
      dates: null,
      prices: null,
      closePrices: null,
      minValue: null,
      maxValue: null,
      collection: null
    }
  },
  created(){
    this.$on('search', function(term){
      this.search(term);
    })
  },
  computed: {
    symbol: function(){
      return this.data["Meta Data"]["2. Symbol"]
    }
  },
  methods: {
    search: function(term){
      var vm = this;
      vm.loading = true;
      Axios.get(`${API_URL}&symbol=${term}`)
      .then(data => {
        this.data = data.data
        if ('Error Message' in this.data){
          console.log("throwing");
          throw this.data["Error Message"];
        }
        this.getValues(vm.data)
        this.loading = false;
      })
      .catch(err => console.error(err))
    },
    getValues(data){
      /*
        Order:
            "[0] open": "162.6200",
            "[1] high": "165.4200",
            "[2] low": "162.4100",
            "[3] close": "163.6500",
            "[4] volume": "28289920"
      */

      var prices=[]
      var closePrices=[]
      var dates=[]
      var collection=[];
      var timeSeries = data["Time Series (Daily)"];

      for (var key in timeSeries){
        prices.push(_.values(timeSeries[key]))
        closePrices.push(timeSeries[key]["4. close"])
        dates.push(key)
        collection.push([key, timeSeries[key]["4. close"]])
      }

      this.dates = dates.reverse();
      this.prices = prices.reverse();
      this.closePrices = closePrices.reverse();
      this.collection = collection.reverse();
      this.minValue = _.min(closePrices);
      this.maxValue = _.max(closePrices);
      
    }
  }
}
</script>

<style>
@import '/assets/css/styles.css';
</style>