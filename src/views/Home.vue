<template>
  <div class="home-container">
    <el-card class="welcome-card" shadow="hover">
      <template #header>
        <div class="card-header flex-between">
          <span>📱 Phone Manage</span>
          <el-tag type="success">Vue 3 + TS</el-tag>
        </div>
      </template>
      <p class="welcome-text">
        欢迎使用 Phone Manage 管理系统
      </p>
      <el-row :gutter="20" class="tech-stack">
        <el-col :span="6" v-for="tech in techStack" :key="tech.name">
          <el-card shadow="hover" class="tech-card">
            <div class="flex-col-center">
              <span class="tech-icon">{{ tech.icon }}</span>
              <span class="tech-name">{{ tech.name }}</span>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </el-card>

    <el-card class="chart-card" shadow="hover">
      <template #header>
        <span>📊 ECharts 示例</span>
      </template>
      <v-chart class="chart" :option="chartOption" autoresize />
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import VChart from 'vue-echarts'
import type { ComposeOption } from 'echarts/core'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { BarChart } from 'echarts/charts'
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  LegendComponent,
} from 'echarts/components'
import type { BarSeriesOption } from 'echarts/charts'
import type {
  TitleComponentOption,
  TooltipComponentOption,
  GridComponentOption,
  LegendComponentOption,
} from 'echarts/components'

// 按需注册 ECharts 组件
use([
  CanvasRenderer,
  BarChart,
  TitleComponent,
  TooltipComponent,
  GridComponent,
  LegendComponent,
])

type ECOption = ComposeOption<
  | BarSeriesOption
  | TitleComponentOption
  | TooltipComponentOption
  | GridComponentOption
  | LegendComponentOption
>

const techStack = ref([
  { name: 'Vue 3', icon: '💚' },
  { name: 'Element Plus', icon: '🎨' },
  { name: 'ECharts', icon: '📊' },
  { name: 'UnoCSS', icon: '🎯' },
])

const chartOption = ref<ECOption>({
  title: {
    text: '手机品牌销量统计',
    left: 'center',
  },
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'shadow',
    },
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true,
  },
  xAxis: {
    type: 'category',
    data: ['华为', '小米', 'OPPO', 'vivo', '苹果', '三星'],
  },
  yAxis: {
    type: 'value',
  },
  series: [
    {
      name: '销量',
      type: 'bar',
      data: [320, 280, 250, 230, 350, 150],
      itemStyle: {
        borderRadius: [4, 4, 0, 0],
        color: {
          type: 'linear',
          x: 0, y: 0, x2: 0, y2: 1,
          colorStops: [
            { offset: 0, color: '#409EFF' },
            { offset: 1, color: '#79BBFF' },
          ],
        },
      },
    },
  ],
})
</script>

<style scoped>
.home-container {
  padding: 20px;
}

.welcome-card {
  margin-bottom: 20px;
}

.card-header {
  font-size: 18px;
  font-weight: bold;
}

.welcome-text {
  font-size: 16px;
  color: #666;
  margin-bottom: 20px;
}

.tech-stack {
  margin-top: 10px;
}

.tech-card {
  text-align: center;
  cursor: pointer;
  transition: transform 0.3s;
}

.tech-card:hover {
  transform: translateY(-4px);
}

.tech-icon {
  font-size: 32px;
  margin-bottom: 8px;
}

.tech-name {
  font-size: 14px;
  color: #333;
  font-weight: 500;
}

.chart-card {
  margin-bottom: 20px;
}

.chart {
  height: 400px;
  width: 100%;
}
</style>
