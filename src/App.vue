<template>
  <div class="container">
    <add-ticker
      @add-ticker-from-inner="add"
      :arrTickers="tickers"
      :disabled="tooManyTickersAdded"
    ></add-ticker>
    <div class="main-content" v-if="tickers.length !== 0">
      <div class="filter-block">
        <div>
          <button
            @click="page = page - 1"
            v-if="page > 1"
          >Назад
          </button>
          <button
            @click="page = page + 1"
            v-if="hasNextPage"
          >Вперёд
          </button>
        </div>
        <div>Фильтр:
          <input type="text"
                 v-model="filter"
          >
        </div>

      </div>
      <div class="cards">
        <div class="card-item"
             :class="{
                'active': selectedTicker === t
             }"
             v-for="t in paginatedTickers"
             :key="t.name"
             @click="select(t)"
        >
          <div class="card-item__label">
            <span class="name">{{ t.name }}</span>
            <span class="line"> - </span>
            <span class="currency">USD</span>
          </div>
          <div class="card-item__price">{{ formatPrice(t.price) }}</div>
          <button class="card-item__remove"
                  @click.stop="remove(t)"
          >
            <svg fill="none" height="24" stroke="#000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                 viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
              <polyline points="3 6 5 6 21 6"/>
              <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/>
              <line x1="10" x2="10" y1="11" y2="17"/>
              <line x1="14" x2="14" y1="11" y2="17"/>
            </svg>
            Удалить
          </button>
        </div>
      </div>
      <div class="chart-wrapper" v-if="selectedTicker">
        <div class="chart-header">
          <div class="chart-title">{{ selectedTicker.name }} - USD</div>
          <div class="chart-close" @click="selectedTicker = null">х</div>
        </div>
        <div class="chart"
             ref="graph"
        >
          <div class="chart-item"
               v-for="(bar, idx) in normalizedGraph"
               :key="idx"
               :style="{ height: `${bar}%`, width: graphElementWidth + 'px' }"
               :title="bar"
               ref="graphElement"
          ></div>
        </div>
        <label class="change-column-width">
          <span>Изменить ширину столбца</span>
          <input type="text"
                 v-model="graphElementWidth"
                 title="Ширина столбца"
          >
        </label>

      </div>
    </div>
  </div>
</template>

<script>

import {subscribeToTicker, unsubscribeFromTicker} from "@/api";
import AddTicker from './components/AddTicker'

export default {
  components: {
    AddTicker
  },
  data() {
    return {
      filter: '',
      tickers: [],
      selectedTicker: null,
      graph: [],
      // recList: {},
      // allTickersObj: {},
      // tickerAlreadyHave: false,
      page: 1,
      maxGraphElements: 1,
      graphElementWidth: 5,
    }
  },
  created() {
    const windowData = Object.fromEntries(new URL(window.location).searchParams.entries())

    if (windowData.filter) {
      this.filter = windowData.filter
    }

    if (windowData.page) {
      this.page = windowData.page
    }

    const tickersData = localStorage.getItem('cryptonomicon-list')

    const graphEl = localStorage.getItem('cryptonomicon-column')

    if (tickersData) {
      this.tickers = JSON.parse(tickersData)
      this.graphElementWidth = JSON.parse(graphEl)

      this.tickers.forEach(ticker => {
        subscribeToTicker(ticker.name, newPrice => {
          this.updateTicker(ticker.name, newPrice)
        });
      })
    }
  },
  // beforeMount() {
  //   let loadingNames = async () => {
  //     const fetchNames = await fetch('https://min-api.cryptocompare.com/data/all/coinlist?summary=true');
  //     this.allTickersObj = await fetchNames.json();
  //     this.allTickersObj = this.allTickersObj.Data;
  //   }
  //   loadingNames()
  // },
  mounted() {
    window.addEventListener('resize', this.calculateMaxGraphElements)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.calculateMaxGraphElements)
  },
  computed: {
    tooManyTickersAdded() {
      return this.tickers.length > 4;
    },
    startIndex() {
      return (this.page - 1) * 6
    },
    endIndex() {
      return this.page * 6
    },
    filteredTickers() {
      return this.tickers.filter(ticker => ticker.name.includes(this.filter));
    },
    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex)
    },
    hasNextPage() {
      return this.filteredTickers.length > this.endIndex
    },
    normalizedGraph() {
      const maxValue = Math.max(...this.graph)
      const minValue = Math.min(...this.graph)

      if (maxValue === minValue) {
        return this.graph.map(() => 50)
      }

      return this.graph.map(
        price => 5 + ((price - minValue) * 95 / (maxValue - minValue))
      )
    },
    pageStateOptions() {
      return {
        filter: this.filter,
        page: this.page
      }
    }
  },
  methods: {
    transTickers(tickers) {
      tickers = this.tickers
      return tickers;
    },
    calculateMaxGraphElements() {
      if (!this.$refs.graph) {
        return;
      }
      this.maxGraphElements = this.$refs.graph.clientWidth / this.graphElementWidth
    },

    updateTicker(tickerName, price) {
      // this.calculateMaxGraphElements()
      this.tickers
        .filter(t => t.name === tickerName)
        .forEach(t => {
          if (t === this.selectedTicker) {
            this.graph.push(price);
            if (this.graph.length > this.maxGraphElements) {
              this.graph = this.graph.slice(this.graph.length - this.maxGraphElements)
            }
          }
          t.price = price;
        });
    },

    formatPrice(price) {
      if (price === '-') {
        return price
      }
      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },
    add(ticker) {
      const currentTicker = {
        name: ticker,
        price: '-'
      }

      this.filter = ''

      if (ticker !== '') {
        // this.tickers.push(currentTicker)
        this.tickers = [...this.tickers, currentTicker]
        this.ticker = '';
        this.filter = '';
        subscribeToTicker(currentTicker.name, newPrice => {
          this.updateTicker(currentTicker.name, newPrice)
        });
        // this.updateTickers(currentTicker.name)
      }
      return this.tickers
    },
    select(ticker) {
      this.selectedTicker = ticker;
    },
    remove(el) {
      this.tickers = this.tickers.filter(tickersElement => tickersElement !== el)
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers))

      if (this.selectedTicker === el) {
        this.selectedTicker = null
      }
      unsubscribeFromTicker(el.name)

    },
  },
  watch: {
    tickers() {
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers))
    },
    selectedTicker() {
      this.graph = [];
      this.$nextTick().then(this.calculateMaxGraphElements)
    },
    paginatedTickers() {
      if (this.paginatedTickers.length === 0 && this.page > 1) {
        this.page -= 1
      }
    },
    graphElementWidth() {
      localStorage.setItem('cryptonomicon-column', JSON.stringify(this.graphElementWidth))
      this.calculateMaxGraphElements();
    },
    filter() {
      this.page = 1;
    },
    pageStateOptions(value) {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${value.filter}&page=${value.page}`
      )
    },

  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;900&display=swap');
</style>
<style src="./main.css"></style>
