<script setup lang="ts">
import Chart from 'chart.js/auto';
import { onMounted, ref, computed } from 'vue';

let stockHistoryPrices: number[] = [];
let stockHistoryDates: string[] = [];

let stockSymbol = ref('IBM');
const apiKey = import.meta.env.VITE_API_KEY;
let apiUrl = computed(() => {
  return `https://www.alphavantage.co/query?function=TIME_SERIES_MONTHLY_ADJUSTED&symbol=${stockSymbol.value}&apikey=${apiKey}`;
})

onMounted(() => {
  const ctx = document.getElementById('myChart') as HTMLCanvasElement | null;
  if (ctx) {
    const createChartWithDate = async () => {
      try {
        // fetch stock price data and push to array
        const response = await fetch(apiUrl.value);
        const data = await response.json();

        console.log(data);
        console.log(data['Meta Data']['2. Symbol']);
        console.log(Object.keys(data['Monthly Adjusted Time Series'])[0]);
        console.log(data['Monthly Adjusted Time Series']['1999-12-31']);

        stockHistoryDates = [];
        stockHistoryPrices = [];

        for (let property in data['Monthly Adjusted Time Series']) {
          const formattedStockHistoryDate = `${property.split('-')[1]}/${property.split('-')[2]}/${property.split('-')[0]}`
          stockHistoryDates.unshift(formattedStockHistoryDate);

          stockHistoryPrices.unshift(Number(data['Monthly Adjusted Time Series'][property]['4. close']));
        }

        // create chart with fetched data
        new Chart(ctx, {
          type: 'line',
          data: {
            labels: stockHistoryDates,
            datasets: [{
              label: 'Stock Market Price',
              data: stockHistoryPrices,
              borderWidth: 1,
              borderColor: 'red'
            }]
          },
          options: {
            scales: {
              y: {
                beginAtZero: true
              }
            },
            elements: {
              point: {
                pointStyle: false
              }
            },
            plugins: {
              tooltip: {
                callbacks: {
                  label: function(context) {
                    let label = context.dataset.label || '';

                    if (label) {
                      label += ': ';
                    }
                    if (context.parsed.y !== null) {
                      label += new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(context.parsed.y);
                    }
                    return label;
                  }
                }
              }
            }
          }
        });
      } catch (error) {
        console.error('Error fetching data and creating chart', error);
      }
    }
    createChartWithDate();
  }
});
</script>
<template>
  <h2>Chart</h2>
  <canvas id="myChart"></canvas>
</template>
