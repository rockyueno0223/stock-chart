<script setup lang="ts">
import Chart from 'chart.js/auto';
import { onMounted, ref, computed } from 'vue';

let stockHistoryPrices: number[] = [];
let stockHistoryDates: string[] = [];

const apiKey = import.meta.env.VITE_API_KEY;

let inputStockSymbol = ref('IBM');
let activeStockSymbol = ref('');
let dateRange = ref('max')

let apiUrl = computed(() => {
  if (dateRange.value === 'max' || dateRange.value === '5years') {
    return `https://www.alphavantage.co/query?function=TIME_SERIES_MONTHLY_ADJUSTED&symbol=${activeStockSymbol.value}&apikey=${apiKey}`;
  } else if (dateRange.value === '1year') {
    return `https://www.alphavantage.co/query?function=TIME_SERIES_WEEKLY_ADJUSTED&symbol=${activeStockSymbol.value}&apikey=${apiKey}`;
  } else if(dateRange.value === '1month'){
    return `https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=${activeStockSymbol.value}&interval=60min&outputsize=full&extended_hours=false&apikey=${apiKey}`;
  } else {
    return '';
  }
})

let chartInstance: Chart | null = null;

const createChartWithDate = async (inputValue: string) => {
  const ctx = document.getElementById('myChart') as HTMLCanvasElement | null;
  if (ctx) {
    try {
      // destroy existing chart
      if (chartInstance) {
        chartInstance.destroy();
      }

      // fetch stock price data and push to array
      activeStockSymbol.value = inputStockSymbol.value;
      dateRange.value = inputValue;

      const response = await fetch(apiUrl.value);
      const data = await response.json();

      console.log(data);

      let sortedData: { [key: string] : any } = {};

      if (dateRange.value === 'max') {
        sortedData = data['Monthly Adjusted Time Series'];
      } else if (dateRange.value === '5years') {
        const monthsInFiveYears: number = 60;
        let counter = 0;
        for (let property in data['Monthly Adjusted Time Series']) {
          if (counter < monthsInFiveYears) {
            sortedData[property] = data['Monthly Adjusted Time Series'][property];
          } else {
            break;
          }
          counter++
        }
      } else if (dateRange.value === '1year') {
        const weeksInOneYear: number = 52;
        let counter = 0;
        for (let property in data['Weekly Adjusted Time Series']) {
          if (counter < weeksInOneYear) {
            sortedData[property] = data['Weekly Adjusted Time Series'][property];
          } else {
            break;
          }
          counter++
        }
      } else if (dateRange.value === '1month') {
        sortedData = data['Time Series (60min)'];
      }

      stockHistoryDates = [];
      stockHistoryPrices = [];

      for (let property in sortedData) {
        let formattedStockHistoryDate = '';
        if (dateRange.value === '1month') {
          const propertyDate = property.split(' ')[0];
          const propertyTime = property.split(' ')[1];
          formattedStockHistoryDate = `${propertyDate.split('-')[1]}/${propertyDate.split('-')[2]} ${propertyTime.split(':')[0]}:${propertyTime.split(':')[1]}`;
        } else {
          formattedStockHistoryDate = `${property.split('-')[1]}/${property.split('-')[2]}/${property.split('-')[0]}`;
        }
        stockHistoryDates.unshift(formattedStockHistoryDate);

        stockHistoryPrices.unshift(Number(sortedData[property]['4. close']));
      }

      // create chart with fetched data
      chartInstance = new Chart(ctx, {
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
              beginAtZero: false
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
                label: function (context) {
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
};

onMounted(() => {
  createChartWithDate('max');
});
</script>

<template >
  <div class="w-full flex flex-col justify-center">
    <h2>Chart</h2>
    <form action="" id="search-form" class="w-full">
      <input type="text" name="search-form-symbol" id="search-form-symbol" v-model="inputStockSymbol" class="border border-black rounded">
      <div class="w-full flex justify-center">
        <button type="button" @click="createChartWithDate('1month')" class="border border-black rounded mx-1 px-2">1 Month</button>
        <button type="button" @click="createChartWithDate('1year')" class="border border-black rounded mx-1 px-2">1 Year</button>
        <button type="button" @click="createChartWithDate('5years')" class="border border-black rounded mx-1 px-2">5 Years</button>
        <button type="button" @click="createChartWithDate('max')" class="border border-black rounded mx-1 px-2">Max</button>
      </div>
    </form>
    <p>Stock Symbol: {{ activeStockSymbol }}</p>
    <p>date range: {{ dateRange }}</p>
    <p>api url: {{ apiUrl }}</p>
    <canvas id="myChart"></canvas>
  </div>
</template>
