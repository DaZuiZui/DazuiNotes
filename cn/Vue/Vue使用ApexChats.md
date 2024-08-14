# Vue使用ApexChats

~~~java
npm install apexcharts vue-apexcharts
~~~

## 引入

~~~java
import Vue from 'vue'
import VueApexCharts from 'vue-apexcharts'

Vue.component('apexchart', VueApexCharts)
~~~

## Code

~~~java
<template>
  <div>
    <apexchart type="line" :options="chartOptions" :series="series"></apexchart>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 数据序列
      series: [
        {
          name: 'Sample Data',
          data: [10, 20, 30, 40, 50, 60]
        }
      ],
      // 图表选项
      chartOptions: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun']
        }
      }
    };
  }
};
</script>

~~~

