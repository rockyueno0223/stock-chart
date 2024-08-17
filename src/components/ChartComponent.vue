<script setup lang="ts">
import Chart from 'chart.js/auto';
import { onMounted } from 'vue';

onMounted(() => {
  const ctx = document.getElementById('myChart') as HTMLCanvasElement | null;
  if (ctx) {
    new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
        datasets: [{
          label: '# of Votes',
          data: [12, 19, 3, 5, 2, 3],
          borderWidth: 1
        }]
      },
      options: {
        scales: {
          y: {
            beginAtZero: true
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
