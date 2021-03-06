<template>
  <div>
    <div class="flex justify-between px-10 py-8">
      <h2 class="text-white m-0 text-xl font-normal">Price in {{ currencyName }}</h2>
      <div>
        <button @click="period('day')"  :class="{ 'bg-blue-darker text-white': type === 'day' }" class="py-2 px-4 bg-transparent hover:bg-blue-darker rounded-md hover:text-white text-blue-light font-normal text-xs">Day</button>
        <button @click="period('week')"  :class="{ 'bg-blue-darker text-white': type === 'week' }" class="py-2 px-4 bg-transparent hover:bg-blue-darker rounded-md hover:text-white text-blue-light font-normal text-xs">Week</button>
        <button @click="period('month')"  :class="{ 'bg-blue-darker text-white': type === 'month' }" class="py-2 px-4 bg-transparent hover:bg-blue-darker rounded-md hover:text-white text-blue-light font-normal text-xs">Month</button>
        <button @click="period('quarter')"  :class="{ 'bg-blue-darker text-white': type === 'quarter' }" class="py-2 px-4 bg-transparent hover:bg-blue-darker rounded-md hover:text-white text-blue-light font-normal text-xs">Quarter</button>
        <button @click="period('year')"  :class="{ 'bg-blue-darker text-white': type === 'year' }" class="py-2 px-4 bg-transparent hover:bg-blue-darker rounded-md hover:text-white text-blue-light font-normal text-xs">Year</button>
      </div>
    </div>

    <price-chart :chartData="chartData" :options="options" :height="314"></price-chart>
  </div>
</template>

<script type="text/ecmascript-6">
import CryptoCompareService from '@/services/crypto-compare'
import PriceChart from '@/components/charts/price-chart'
import { mapGetters } from 'vuex'
import store from '@/store'

export default {
  components: {PriceChart},

  data: () => ({
    type: 'day',
    chartData: {},
    options: {
      showScale: true,
      responsive: true,
      maintainAspectRatio: false,
      legend: {
        display: false,
      },
      layout: {
        padding: {
          left: 50,
          right: 50,
          top: 0,
          bottom: 50,
        }
      },
      scales: {
        yAxes: [
          {
            ticks: {
              callback: (value, index, values) => {
                // Skip every second tick
                if (index % 2 === 0) return

                if ([store.getters['network/token'], 'BTC', 'ETH', 'LTC'].some(c => store.getters['currency/name'].indexOf(c) > -1)) {
                  return store.getters['currency/symbol'] + value.toFixed(8)
                }

                return store.getters['currency/symbol'] + value.toFixed(2)
              },
              fontColor: '#838a9b',
              fontSize: 13,
            },
            gridLines: {
              color: '#282b38',
            },
          },
        ],
        xAxes: [
          {
            // type: 'time',
            // time: {
            //   unit: 'day',
            //   unitStepSize: 1,
            //   displayFormats: {
            //     day: 'MMM D',
            //   },
            // },
            gridLines: {
              display: true,
              color: '#282b38',
            },
            ticks: {
              fontColor: '#838a9b',
              fontSize: 13,
            },
          },
        ],
      },
      tooltips: {
        backgroundColor: '#272936',
        titleFontStyle: 'normal',
        titleFontSize: 18,
        bodyFontFamily: "'Proxima Nova', sans-serif",
        cornerRadius: 3,
        bodyFontColor: '#838a9b',
        bodyFontSize: 14,
        xPadding: 14,
        yPadding: 14,
        displayColors: false,
        mode: 'index',
        intersect: false,
        // borderWidth: 1,
        // borderColor: '#037cff',
        callbacks: {
          title: tooltipItem => {
            const name = store.getters['currency/name']
            const symbol = store.getters['currency/symbol']

            return `${symbol} ${tooltipItem[0].yLabel} ${name}`
          },
          label: tooltipItem => ''
          // label: tooltipItem => `BTC ${tooltipItem.yLabel}`
        }
      }
    }
  }),

  computed: {
    ...mapGetters('currency', { currencyName: 'name' }),
  },

  mounted() {
    this.prepareComponent()
  },

  methods: {
    prepareComponent() {
      this.renderChart()

      this.watchCurrencyName()
      this.watchNetworkToken()
    },

    period(type) {
      this.type = type

      this.renderChart()
    },

    renderChart(type) {
      CryptoCompareService[this.type]().then(response => {
        this.chartData = {
          labels: response.labels,
          datasets: [{
            type: 'line',
            pointHoverBackgroundColor: '#fff',
            borderColor: '#535972',
            pointHoverBorderColor: '#037cff',
            pointBackgroundColor: 'rgba(0,0,0,0)',
            pointBorderColor: 'rgba(0,0,0,0)',
            pointHoverRadius: 5,
            pointHoverBorderWidth: 4,
            fill: false,
            // data: this.chartData.map((point, index) => {
            //   return {
            //     t: this.labels[index],
            //     y: point,
            //   }
            // }),
            data: response.datasets
          }],
        }
      })
    },

    watchCurrencyName() {
      this.$store.watch((state) => state.currency.name, (value) => this.renderChart())
    },

    watchNetworkToken() {
      this.$store.watch((state) => state.network.token, (value) => this.renderChart())
    },
  }
}
</script>
