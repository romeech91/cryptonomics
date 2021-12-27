<template>
  <div class="header">
    <label>
      <span>Тикер</span>
      <input type="text" placeholder="Например BTC"
             v-model="ticker"
             @keydown.enter="add(arrTickers)"
             @input="recommend"
      >
    </label>
    <div class="recommends" v-if="recList.length > 0">
        <span
          v-for="item in recList"
          :key="item"
          @click="addToDesc(item, arrTickers)"
        >{{ item }}</span>
    </div>
    <div class="ticker-error" v-if="tickerAlreadyHave">Такой тикер уже добавлен!</div>
    <add-button
      @click.native="add(arrTickers)"
      :disabled="disabled"
    ></add-button>
  </div>
</template>

<script>
import AddButton from './AddButton.vue'
// import {subscribeToTicker} from "@/api";

export default {
  data() {
    return {
      ticker: '',
      tickers: [],
      recList: {},
      tickerAlreadyHave: false,
      allTickersObj: {},
    }
  },

  components: {
    AddButton
  },

  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false
    },
    arrTickers: {
      type: Array,
      required: true
    }
    // currentTickers: {
    //   type: Array
    // }
  },

  emits: {
    // 'add-ticker': value => typeof value === 'string' && value.length > 0
  },

  beforeMount() {
    let loadingNames = async () => {
      const fetchNames = await fetch('https://min-api.cryptocompare.com/data/all/coinlist?summary=true');
      this.allTickersObj = await fetchNames.json();
      this.allTickersObj = this.allTickersObj.Data;
    }
    loadingNames()
  },
  created() {

  },
  methods: {
    add(arr) {
      if (arr.find(t => t.name === this.ticker)) {
        this.tickerAlreadyHave = true
        return
      }

      if (this.ticker.length === 0) {
        return;
      }
      this.$emit('add-ticker-from-inner', this.ticker)
      this.ticker = '';
    },

    recommend() {
      this.tickerAlreadyHave = false
      const myObj = this.allTickersObj;

      if (this.ticker.length !== 0) {
        let filtered = Object.values(myObj).filter(item => item['FullName'].includes(this.ticker));
        let formattedArr = []
        filtered.forEach(obj => {
          formattedArr.push(obj['Symbol'])
        })
        this.recList = Array.from(Object.values(formattedArr))
        this.recList.splice(4)
      } else {
        this.recList.splice(0)
      }
    },
    addToDesc(elem, arr) {
      if (arr.find(t => t.name === elem)) {
        this.tickerAlreadyHave = true
      } else {
        this.ticker = elem
        this.add(arr)
        this.tickerAlreadyHave = false
      }
    },
  },
}
</script>