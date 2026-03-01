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
        <div class="card-header flex-between">
          <span>📈 新品旗舰销量排行榜</span>
        </div>
      </template>
      <div v-if="currentWeek" class="week-badge">
        <div class="week-badge__year">{{ currentWeek.split('-W')[0] }}</div>
        <div class="week-badge__week">W{{ currentWeek.split('-W')[1] }}</div>
      </div>
      <v-chart class="chart" :option="chartOption" autoresize />
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import VChart from 'vue-echarts'
import type { ComposeOption } from 'echarts/core'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { BarChart } from 'echarts/charts'
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  ToolboxComponent,
} from 'echarts/components'
import type { BarSeriesOption } from 'echarts/charts'
import type {
  TitleComponentOption,
  TooltipComponentOption,
  GridComponentOption,
  ToolboxComponentOption,
} from 'echarts/components'
import flagshipData from '@/assets/flagship.json'

// 按需注册 ECharts 组件
use([
  CanvasRenderer,
  BarChart,
  TitleComponent,
  TooltipComponent,
  GridComponent,
  ToolboxComponent,
])

type ECOption = ComposeOption<
  | BarSeriesOption
  | TitleComponentOption
  | TooltipComponentOption
  | GridComponentOption
  | ToolboxComponentOption
>

const techStack = ref([
  { name: 'Vue 3', icon: '💚' },
  { name: 'Element Plus', icon: '🎨' },
  { name: 'ECharts', icon: '📊' },
  { name: 'UnoCSS', icon: '🎯' },
])

type FlagshipEntry = {
  week: string
  sales: number
  tag: string
}

type FlagshipSeries = {
  brand: string
  series: string
  entries: FlagshipEntry[]
}

type FlagshipFile = {
  flagship: FlagshipSeries[]
}

const data = flagshipData as unknown as FlagshipFile

const weekKey = (week: string) => {
  const m = /^(\d{4})-W(\d{1,2})$/.exec(week)
  if (!m)
    return Number.NEGATIVE_INFINITY
  return Number(m[1]) * 100 + Number(m[2])
}

const weeks = computed(() => {
  const set = new Set<string>()
  for (const s of data.flagship) {
    for (const e of s.entries)
      set.add(e.week)
  }
  return [...set].sort((a, b) => weekKey(a) - weekKey(b))
})

const currentWeekIndex = ref(0)
const currentWeek = computed(() => weeks.value[currentWeekIndex.value] ?? '')

watch(weeks, (w) => {
  if (!w.length) {
    currentWeekIndex.value = 0
    return
  }
  currentWeekIndex.value = w.length - 1
}, { immediate: true })

const playIntervalMs = 1200
let timer: number | undefined

watch(currentWeek, (w) => {
  if (!w) {
    if (timer != null) {
      window.clearInterval(timer)
      timer = undefined
    }
    return
  }
  if (timer != null)
    return
  timer = window.setInterval(() => {
    const wList = weeks.value
    if (!wList.length)
      return
    currentWeekIndex.value = (currentWeekIndex.value + 1) % wList.length
  }, playIntervalMs)
}, { immediate: true })

onUnmounted(() => {
  if (timer != null)
    window.clearInterval(timer)
})

const brandGradient = (brand: string) => {
  const map: Record<string, [string, string]> = {
    苹果: ['#A1A1AA', '#f68640'],
    小米: ['#FF7A18', '#FF3D00'],
    OPPO: ['#4ADE80', '#16A34A'],
    vivo: ['#60A5FA', '#2563EB'],
    荣耀: ['#F472B6', '#DB2777'],
    华为: ['#FB7185', '#E11D48'],
  }
  const [from, to] = map[brand] ?? ['#A78BFA', '#6D28D9']
  return {
    type: 'linear' as const,
    x: 0,
    y: 0,
    x2: 1,
    y2: 0,
    colorStops: [
      { offset: 0, color: from },
      { offset: 1, color: to },
    ],
  }
}

const chartOption = computed<ECOption>(() => {
  const week = currentWeek.value
  const rows = data.flagship
    .map((s) => {
      const entry = s.entries.find(e => e.week === week)
      if (!entry)
        return null
      return {
        name: `${s.brand} ${s.series}`,
        sales: entry.sales,
        brand: s.brand,
      }
    })
    .filter((v): v is { name: string; sales: number; brand: string } => Boolean(v))
    .sort((a, b) => b.sales - a.sales)

  const names = rows.map(r => r.name)
  const values = rows.map(r => r.sales)

  return {
    title: {
      text: '新品旗舰销量排行榜（Sell out）',
      subtext: week ? `单位：万` : '',
      left: 'center',
    },
    toolbox: {
      right: 10,
      feature: {
        saveAsImage: { show: true },
      },
    },
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow',
      },
      valueFormatter: (value: unknown) => `${value} 万`,
    },
    grid: {
      left: 160,
      right: 50,
      bottom: 40,
      top: 80,
    },
    xAxis: {
      type: 'value',
      axisLabel: {
        formatter: '{value} 万',
      },
      splitLine: {
        lineStyle: {
          color: 'rgba(0,0,0,0.06)',
        },
      },
    },
    yAxis: {
      type: 'category',
      inverse: true,
      data: names,
      axisLabel: {
        color: '#111827',
        width: 150,
        overflow: 'truncate',
      },
    },
    series: [
      {
        name: '销量',
        type: 'bar',
        universalTransition: true,
        barWidth: 18,
        label: {
          show: true,
          position: 'right',
          color: '#111827',
          valueAnimation: true,
          formatter: (p: { value?: unknown }) => `${p.value ?? ''} 万`,
        },
        data: values.map((v, i) => ({
          value: v,
          itemStyle: {
            borderRadius: [0, 8, 8, 0],
            color: brandGradient(rows[i]?.brand ?? ''),
          },
        })),
        emphasis: {
          focus: 'series',
        },
      },
    ],
    animationDurationUpdate: 900,
    animationEasingUpdate: 'cubicOut',
  }
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
  position: relative;
}

.chart {
  height: 520px;
  width: 100%;
}

.week-badge {
  position: absolute;
  right: 18px;
  top: 50%;
  transform: translateY(-50%);
  width: 64px;
  height: 64px;
  border-radius: 999px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(239, 68, 68, 0.55);
  color: #fff;
  font-weight: 700;
  font-size: 16px;
  letter-spacing: 0.5px;
  box-shadow: 0 14px 30px rgba(239, 68, 68, 0.28);
  backdrop-filter: blur(6px);
  user-select: none;
  pointer-events: none;
}

.week-badge__year {
  font-size: 11px;
  line-height: 1;
  opacity: 0.92;
  letter-spacing: 0.4px;
}

.week-badge__week {
  margin-top: 3px;
  font-size: 16px;
  line-height: 1;
  letter-spacing: 0.6px;
}
</style>
