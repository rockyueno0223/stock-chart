<script setup lang="ts">
import Chart from 'chart.js/auto';
import { onMounted, ref, computed } from 'vue';

onMounted(() => {
  const ctx = document.getElementById('myChart') as HTMLCanvasElement | null;
  if (ctx) {
    new Chart(ctx, {
      type: 'line',
      data: {
        labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange', 'Pink'],
        datasets: [{
          label: '# of Votes',
          data: [12, 19, 3, 5, 2, 3, 8],
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

    let stockSymbol = ref('IBM');
    const apiKey = import.meta.env.VITE_API_KEY;
    let apiUrl = computed(() => {
      return `https://www.alphavantage.co/query?function=TIME_SERIES_MONTHLY&symbol=${stockSymbol.value}&apikey=${apiKey}`;
    })

    const createChartWithDate = async (dateRange: string) => {
      const response = await fetch(apiUrl.value);
      const data = await response.json();
      console.log(data['Meta Data']['2. Symbol']);
      console.log(data['Monthly Time Series']['1999-12-31']);
    }
    createChartWithDate("");
  }
});
</script>
<template>
  <h2>Chart</h2>
  <canvas id="myChart"></canvas>
</template>
