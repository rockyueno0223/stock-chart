<script setup lang="ts">
import Chart from 'chart.js/auto';
import { onMounted } from 'vue';

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
  }
});
</script>
<template>
  <h2>Chart</h2>
  <canvas id="myChart"></canvas>
</template>
