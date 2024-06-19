<template>
  <div class="chart-container">
    <h2>Options Strategy Risk & Reward Analysis</h2>
    <canvas ref="chartCanvas"></canvas>
    <div v-if="maxProfit !== null" class="results">
      <p><strong>Max Profit:</strong> {{ maxProfit }}</p>
      <p><strong>Max Loss:</strong> {{ maxLoss }}</p>
      <p><strong>Break Even Points:</strong> {{ breakEvenPoints.length > 0 ? breakEvenPoints.join(', ') : 'None' }}</p>
    </div>
  </div>
</template>

<script>
import {
  Chart as ChartJS,
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  Title,
  CategoryScale,
  Tooltip,
  Legend
} from 'chart.js';

ChartJS.register(
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  Title,
  CategoryScale,
  Tooltip,
  Legend
);

export default {
  name: 'CodingChallenge',
  props: {
    optionsData: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      maxProfit: null,
      maxLoss: null,
      breakEvenPoints: [],
      chart: null
    };
  },
  mounted() {
    this.calculateRiskReward();
    this.createChart();
  },
  methods: {
    calculateRiskReward() {
      const results = this.optionsData.map(option => {
        const profitLoss = option.long_short === 'long' ? option.bid : -option.ask;
        return {
          strike_price: option.strike_price,
          profitLoss
        };
      });

      this.maxProfit = Math.max(...results.map(res => res.profitLoss));
      this.maxLoss = Math.min(...results.map(res => res.profitLoss));
      this.breakEvenPoints = this.calculateBreakEvenPoints(results);
    },
    calculateBreakEvenPoints(results) {
      // TODO: Not sure about this logic
      const breakEvenPoints = [];

      for (let i = 1; i < results.length; i++) {
        const prev = results[i - 1];
        const curr = results[i];
        if (prev.profitLoss * curr.profitLoss < 0) {
          const breakEven = prev.strike_price + (curr.strike_price - prev.strike_price) * (0 - prev.profitLoss) / (curr.profitLoss - prev.profitLoss);
          breakEvenPoints.push(breakEven.toFixed(2));
        }
      }

      return breakEvenPoints;
    },
    createChart() {
      const ctx = this.$refs.chartCanvas.getContext('2d');
      this.chart = new ChartJS(ctx, {
        type: 'line',
        data: {
          labels: this.optionsData.map(option => option.strike_price),
          datasets: [{
            label: 'Profit/Loss',
            borderColor: '#42b983',
            backgroundColor: 'rgba(66, 185, 131, 0.2)',
            data: this.optionsData.map(option => option.long_short === 'long' ? option.bid : -option.ask)
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              title: {
                display: true,
                text: 'Strike Price'
              }
            },
            y: {
              title: {
                display: true,
                text: 'Profit/Loss'
              }
            }
          }
        }
      });
    }
  }
};
</script>

<style scoped>
.chart-container {
  max-width: 600px;
  margin: 20px auto;
  text-align: center;
}

canvas {
  width: 100% !important;
  height: 500px !important; 
  /* TODO: height needs to be handled */
}

.results {
  margin-top: 20px;
}
</style>
