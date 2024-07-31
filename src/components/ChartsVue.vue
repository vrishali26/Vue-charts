<template>
  <div class="container">
    <div id="wrapper">
      <div class="nav-buttons">
        <button @click="prevData()">Prev</button>
        <button @click="nextData()">Next</button>
      </div>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import { fetchCandlestickData } from '../service';

export default {
  name: 'ChartsVue',
  data: () => ({
    index : 0,    
    open : [],
    high : [],
    low : [],
    close : [],
    volume : [],
    chunkSize : 50,
    chunks : [],
    graphData : [],
    myChart : Chart,
  }),
  methods: {
    nextData() {
      if (this.index < this.graphData.length) {
        this.index++;
        console.log(this.index);
        this.myChart.destroy();
        this.updateChart();
      }
    },
    prevData() {
      if (this.index > 0) {
        this.index--;
        console.log(this.index);
        this.myChart.destroy();
        this.updateChart();
      }
    },
    updateChart() {
      const newCanvas = document.createElement("canvas");
      newCanvas.setAttribute("id", "myChart");
      const wrapper = document.getElementById("wrapper");
      wrapper.appendChild(newCanvas);
      console.log(this.graphData.length);
      this.open = [];
      this.high = [];
      this.low = [];
      this.close = [];
      this.volume = [];
      for (let i = 0; i < this.graphData[this.index].length; i++) {
        this.open.push(Object.values(this.graphData[this.index][i])[0]);
        this.high.push(Object.values(this.graphData[this.index][i])[1]);
        this.low.push(Object.values(this.graphData[this.index][i])[2]);
        this.close.push(Object.values(this.graphData[this.index][i])[3]);
        this.volume.push(Object.values(this.graphData[this.index][i])[4]);
      }
      const ctx = document.getElementById('myChart');
      this.myChart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: this.chunks[this.index],
          datasets: [{
            type: 'line',
            label: 'open',
            data: this.open,
            borderWidth: 1,
            fill: false,
            borderColor: 'rgb(75, 192, 192)',
            tension: 0.1
          }, {
            label: 'low',
            data: this.low,
            backgroundColor: 'rgba(255, 255, 255)',
            borderWidth: 1,
            barThickness: 5,
          }, {
            label: 'high',
            data: this.high,
            backgroundColor: 'rgba(153, 102, 255)',
            borderWidth: 1,
            barThickness: 5,
          }, {
            type: 'line',
            label: 'close',
            data: this.close,
            borderWidth: 1,
            fill: false,
            borderColor: 'rgb(255, 205, 86)',
            tension: 0.1
          }]
        },
        options: {
          scales: {
            x: {
              stacked: true,
            },
            y: {
              beginAtZero: false,
            }
          }
        },
      });
      this.myChart;
    }
  },
  async mounted() {
    var chartData = await fetchCandlestickData();
    const timeSeries = chartData[Object.keys(chartData)[1]];
    var labels = Object.keys(timeSeries);

    for (let i = 0; i < timeSeries.length; i += this.chunkSize) {
      const chunk = timeSeries.slice(i, i + this.chunkSize);
      this.chunks.push(chunk);
      const label = labels.slice(i, i + this.chunkSize);
      labels.push(label);
    }

    for (let i = 0; i < labels.length; i += this.chunkSize) {
      const chunkLabel = labels.slice(i, i + this.chunkSize);
      this.chunks.push(chunkLabel);
      const chunkVal = Object.values(timeSeries).slice(i, i + this.chunkSize);
      this.graphData.push(chunkVal);
    }
    this.updateChart();

  }
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
a {
  color: #42b983;
}
.container {
  width: 80%;
  height: 70%;
  margin: auto;
  overflow: auto;
}
.nav-buttons button {
  display: inline-block;
  padding: .375em 1em;
  background-color: rgb(75, 192, 192);
  border: none;
  border-radius: 5px;
  color: #fff;
  font-weight: bold;
}
.nav-buttons button:first-child {
  float: left;
}
.nav-buttons button:last-child {
  float: right;
}
</style>
