<template>
  <div class="agriculture-container">
    <!-- 数据概览卡片 -->
    <el-row :gutter="20" class="data-overview">
      <el-col :span="4">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>耕地面积</span>
              <el-tag size="small" type="success">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">13,200</div>
            <div class="unit">亩</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="4">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农业产值</span>
              <el-tag size="small" type="success">年度</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">3.2</div>
            <div class="unit">亿元</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="4">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农业合作社</span>
              <el-tag size="small" type="success">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">32</div>
            <div class="unit">个</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="4">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农业人口</span>
              <el-tag size="small" type="success">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">16,200</div>
            <div class="unit">人</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="4">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农机设备数</span>
              <el-tag size="small" type="info">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">185</div>
            <div class="unit">台/套</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="4">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农产品种类数</span>
              <el-tag size="small" type="info">年度</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">24</div>
            <div class="unit">种</div>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 产业分布图表 -->
    <el-row :gutter="20" class="chart-section">
      <el-col :span="12">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农业产业结构分布</span>
            </div>
          </template>
          <div class="chart-container" ref="industryChartRef"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>农业产值趋势</span>
            </div>
          </template>
          <div class="chart-container" ref="trendChartRef"></div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 政策信息 -->
    <el-card class="policy-section" shadow="hover">
      <template #header>
        <div class="card-header">
          <span>农业政策信息</span>
          <el-button type="primary" size="small">发布政策</el-button>
        </div>
      </template>
      <el-table :data="policyList" style="width: 100%">
        <el-table-column prop="title" label="政策标题" min-width="200" />
        <el-table-column prop="type" label="政策类型" width="100" />
        <el-table-column prop="department" label="发布部门" width="150" />
        <el-table-column prop="publishDate" label="发布日期" width="120" />
        <el-table-column prop="status" label="状态" width="100">
          <template #default="{ row }">
            <el-tag :type="row.status === '已生效' ? 'success' : 'warning'">
              {{ row.status }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150">
          <template #default="{ row }">
            <el-button link type="primary" @click="viewPolicy(row)">查看</el-button>
            <el-button link type="primary" @click="editPolicy(row)">编辑</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import * as echarts from 'echarts'

// 图表引用
const industryChartRef = ref<HTMLElement>()
const trendChartRef = ref<HTMLElement>()

// 政策列表数据
const policyList = ref([
  {
    id: 1,
    title: '关于2025年农业补贴政策的通知',
    type: '补贴',
    department: '农业办公室',
    publishDate: '2025-03-15',
    status: '已生效'
  },
  {
    id: 2,
    title: '张庄镇农业产业发展规划（2025-2027）',
    type: '规划',
    department: '经济发展办公室',
    publishDate: '2025-03-10',
    status: '已生效'
  },
  {
    id: 3,
    title: '关于加强农业技术推广工作的通知',
    type: '技术',
    department: '农业技术推广站',
    publishDate: '2025-03-05',
    status: '已生效'
  },
  {
    id: 4,
    title: '关于推进智慧农业建设的指导意见',
    type: '指导',
    department: '信息化办公室',
    publishDate: '2025-04-01',
    status: '已生效'
  },
  {
    id: 5,
    title: '农产品质量安全监管办法',
    type: '监管',
    department: '市场监管所',
    publishDate: '2025-04-10',
    status: '已生效'
  },
  {
    id: 6,
    title: '关于开展农机购置补贴的通知',
    type: '补贴',
    department: '农业办公室',
    publishDate: '2025-04-15',
    status: '已生效'
  },
  {
    id: 7,
    title: '绿色农业发展激励政策',
    type: '激励',
    department: '生态环境办公室',
    publishDate: '2025-04-20',
    status: '已生效'
  },
  {
    id: 8,
    title: '关于加强农产品品牌建设的通知',
    type: '品牌',
    department: '农业办公室',
    publishDate: '2025-04-25',
    status: '已生效'
  }
])

// 初始化产业分布图表
const initIndustryChart = () => {
  if (!industryChartRef.value) return
  const chart = echarts.init(industryChartRef.value)
  const option = {
    title: {
      text: '农业产业结构',
      left: 'center'
    },
    tooltip: {
      trigger: 'item'
    },
    legend: {
      orient: 'vertical',
      left: 'left'
    },
    series: [
      {
        name: '产业分布',
        type: 'pie',
        radius: '50%',
        data: [
          { value: 38, name: '粮食种植' },
          { value: 22, name: '经济作物' },
          { value: 14, name: '蔬菜种植' },
          { value: 10, name: '果树种植' },
          { value: 10, name: '畜牧养殖' },
          { value: 4, name: '水产养殖' },
          { value: 2, name: '其他' }
        ],
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)'
          }
        }
      }
    ]
  }
  chart.setOption(option)
}

// 初始化产值趋势图表
const initTrendChart = () => {
  if (!trendChartRef.value) return
  const chart = echarts.init(trendChartRef.value)
  const option = {
    title: {
      text: '农业产值趋势',
      left: 'center'
    },
    tooltip: {
      trigger: 'axis'
    },
    xAxis: {
      type: 'category',
      data: ['2020', '2021', '2022', '2023', '2024', '2025']
    },
    yAxis: {
      type: 'value',
      name: '亿元'
    },
    series: [
      {
        name: '农业产值',
        type: 'line',
        data: [1.8, 2.0, 2.3, 2.5, 2.8, 3.2],
        smooth: true,
        markPoint: {
          data: [
            { type: 'max', name: '最大值' },
            { type: 'min', name: '最小值' }
          ]
        }
      }
    ]
  }
  chart.setOption(option)
}

// 查看政策详情
const viewPolicy = (row: any) => {
  console.log('查看政策:', row)
}

// 编辑政策
const editPolicy = (row: any) => {
  console.log('编辑政策:', row)
}

onMounted(() => {
  initIndustryChart()
  initTrendChart()
})
</script>

<style scoped>
.agriculture-container {
  padding: 20px;
}

.data-overview {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-content {
  text-align: center;
  padding: 10px 0;
}

.card-content .value {
  font-size: 24px;
  font-weight: bold;
  color: #303133;
}

.card-content .unit {
  font-size: 14px;
  color: #909399;
  margin-top: 5px;
}

.chart-section {
  margin-bottom: 20px;
}

.chart-container {
  height: 400px;
}

.policy-section {
  margin-bottom: 20px;
}

:deep(.el-card__header) {
  padding: 15px 20px;
}

:deep(.el-card__body) {
  padding: 20px;
}

:deep(.el-button--primary.is-link),
:deep(.el-button--primary.is-link:hover) {
  background: none !important;
  border: none !important;
  color: #409EFF !important;
  box-shadow: none !important;
}
</style> 