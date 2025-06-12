<template>
    <div class="dashboard-container">
      <div class="dashboard-header">
        <h1 class="dashboard-title">邹城市张庄镇人民政府云定制化平台</h1>
      </div>
      
      <!-- 顶部统计卡片 -->
      <el-row :gutter="20" class="stat-row">
        <el-col :xs="24" :sm="12" :md="6" v-for="(item, index) in statCards" :key="index">
          <el-card shadow="hover" class="stat-card" :class="item.type">
            <template #header>
              <div class="card-header">
                <span>{{ item.title }}</span>
                <el-tag :type="item.tagType">{{ item.tagText }}</el-tag>
              </div>
            </template>
            <div class="stat-content">
              <el-icon class="stat-icon"><component :is="item.icon" /></el-icon>
              <div class="stat-info">
                <div class="stat-number">{{ item.value }}</div>
                <div class="stat-label">{{ item.label }}</div>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
  
      <!-- 业务系统状态 -->
      <el-card class="system-status" shadow="hover">
        <template #header>
          <div class="card-header">
            <div class="header-left">
              <el-icon><Operation /></el-icon>
              <span>业务系统状态</span>
              <el-tag type="success" effect="dark" class="ml-10">全部正常</el-tag>
            </div>
            <div class="header-actions">
              <el-tooltip content="数据为实时同步">
                <span class="sync-time">最后同步: {{ formatTime(new Date()) }}</span>
              </el-tooltip>
              <el-button type="primary" size="small" :icon="Refresh" @click="refreshSystemStatus">刷新</el-button>
              <el-button type="success" size="small" :icon="Document">报表</el-button>
            </div>
          </div>
        </template>
        
        <el-row :gutter="20" class="system-summary">
          <el-col :xs="24" :sm="12" :md="6">
            <div class="summary-card running">
              <div class="summary-icon"><el-icon><VideoPlay /></el-icon></div>
              <div class="summary-info">
                <div class="summary-title">运行中</div>
                <div class="summary-value">{{ systemList.filter(s => s.status === '运行中').length }}</div>
              </div>
            </div>
          </el-col>
          <el-col :xs="24" :sm="12" :md="6">
            <div class="summary-card maintenance">
              <div class="summary-icon"><el-icon><Tools /></el-icon></div>
              <div class="summary-info">
                <div class="summary-title">维护中</div>
                <div class="summary-value">{{ systemList.filter(s => s.status === '维护中').length }}</div>
              </div>
            </div>
          </el-col>
          <el-col :xs="24" :sm="12" :md="6">
            <div class="summary-card warning">
              <div class="summary-icon"><el-icon><WarningFilled /></el-icon></div>
              <div class="summary-info">
                <div class="summary-title">告警</div>
                <div class="summary-value">{{ systemList.filter(s => s.status === '告警').length }}</div>
              </div>
            </div>
          </el-col>
          <el-col :xs="24" :sm="12" :md="6">
            <div class="summary-card offline">
              <div class="summary-icon"><el-icon><CircleClose /></el-icon></div>
              <div class="summary-info">
                <div class="summary-title">离线</div>
                <div class="summary-value">{{ systemList.filter(s => s.status === '离线').length }}</div>
              </div>
            </div>
          </el-col>
        </el-row>
        
        <el-table 
          :data="systemList" 
          style="width: 100%" 
          :max-height="400"
          border
          stripe
          highlight-current-row
          v-loading="tableLoading"
        >
          <el-table-column prop="systemName" label="系统名称" min-width="180">
            <template #default="{ row }">
              <div class="system-name">
                <el-icon class="system-icon" :class="getSystemIconClass(row)">
                  <component :is="getSystemIcon(row)" />
                </el-icon>
                <span>{{ row.systemName }}</span>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="owner" label="负责人" width="120" />
          <el-table-column prop="status" label="状态" width="100">
            <template #default="{ row }">
              <el-tag 
                :type="getStatusType(row.status)"
                effect="dark"
                size="small"
              >
                {{ row.status }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="updateTime" label="更新时间" width="180" />
          <el-table-column prop="usage" label="使用率" width="180">
            <template #default="{ row }">
              <el-progress 
                :percentage="row.usage || 0"
                :status="row.usage > 80 ? 'exception' : row.usage > 60 ? 'warning' : 'success'"
                :stroke-width="8"
                :format="(p) => `${p}%`"
              />
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150" fixed="right">
            <template #default="{ row }">
              <el-button-group>
                <el-tooltip content="查看详情" placement="top">
                  <el-button type="primary" :icon="View" circle size="small" @click="viewSystemDetail(row)"></el-button>
                </el-tooltip>
                <el-tooltip content="系统设置" placement="top">
                  <el-button type="warning" :icon="Setting" circle size="small" @click="configSystem(row)"></el-button>
                </el-tooltip>
                <el-tooltip content="重启系统" placement="top">
                  <el-button type="danger" :icon="RefreshRight" circle size="small" @click="restartSystem(row)"></el-button>
                </el-tooltip>
              </el-button-group>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
  
      <!-- 硬件资源监控 -->
      <el-row :gutter="20" class="monitor-row">
        <el-col :xs="24" :md="12">
          <el-card class="monitor-card" shadow="hover">
            <template #header>
              <div class="card-header">
                <div class="header-left">
                  <el-icon><TrendCharts /></el-icon>
                  <span>资源使用趋势</span>
                </div>
                <el-radio-group v-model="timeRange" size="small">
                  <el-radio-button label="day">24小时</el-radio-button>
                  <el-radio-button label="week">7天</el-radio-button>
                  <el-radio-button label="month">30天</el-radio-button>
                </el-radio-group>
              </div>
            </template>
            <div ref="trendChartRef" class="chart-container"></div>
          </el-card>
        </el-col>
        <el-col :xs="24" :md="12">
          <el-card class="monitor-card" shadow="hover">
            <template #header>
              <div class="card-header">
                <div class="header-left">
                  <el-icon><Warning /></el-icon>
                  <span>系统告警</span>
                </div>
                <el-button type="primary" size="small" :icon="Refresh" @click="refreshAlarms">刷新</el-button>
              </div>
            </template>
            <el-timeline>
              <el-timeline-item
                v-for="(alarm, index) in alarmList"
                :key="index"
                :type="alarm.type"
                :timestamp="alarm.time"
                :hollow="alarm.type === 'info'"
              >
                <div class="alarm-content">{{ alarm.content }}</div>
              </el-timeline-item>
            </el-timeline>
          </el-card>
        </el-col>
      </el-row>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, reactive, onMounted, onBeforeUnmount, watch, nextTick } from 'vue'
  import * as echarts from 'echarts'
  import { 
    Operation, 
    TrendCharts, 
    Warning, 
    WarningFilled,
    Tickets, 
    HomeFilled, 
    Crop, 
    FirstAidKit, 
    Refresh,
    RefreshRight,
    View,
    Setting,
    Document,
    Tools,
    VideoPlay,
    CircleClose
  } from '@element-plus/icons-vue'
  
  // 统计卡片数据
  const statCards = [
    {
      title: '政务便民服务',
      icon: 'Tickets',
      value: '6',
      label: '系统总数',
      tagText: '运行中',
      tagType: 'success',
      type: 'primary'
    },
    {
      title: '村务管理',
      icon: 'HomeFilled',
      value: '4',
      label: '资源总数',
      tagText: '正常',
      tagType: 'primary',
      type: 'success'
    },
    {
      title: '农业产业信息化',
      icon: 'Crop',
      value: '9TB',
      label: '已用/20TB',
      tagText: '使用中',
      tagType: 'warning',
      type: 'warning'
    },
    {
      title: '应急指挥',
      icon: 'FirstAidKit',
      value: '65%',
      label: 'CPU使用率',
      tagText: '实时',
      tagType: 'info',
      type: 'danger'
    }
  ]
  
  // 统计数据
  const systemStats = ref({
    total: 6,
    running: 18,
    warning: 0,
    error: 0
  })
  
  const hardwareStats = ref({
    total: 4,
    normal: 4,
    warning: 0,
    error: 0
  })
  
  const storageStats = ref({
    total: 20,
    used: 9,
    free: 11
  })
  
  const loadStats = ref({
    cpu: 65,
    memory: 78,
    storage: 45,
    network: 32
  })
  
  // 时间范围选择
  const timeRange = ref('day')
  
  // 系统列表数据
  const systemList = ref([
    { id: 1, systemName: '政务便民服务平台', owner: '王晓强', status: '运行中', updateTime: '2025-05-01 09:00', usage: 78, type: 'public' },
    { id: 2, systemName: '村务管理系统', owner: '李文娜', status: '运行中', updateTime: '2025-05-01 08:30', usage: 65, type: 'village' },
    { id: 3, systemName: '农业信息化平台', owner: '刘志洋', status: '运行中', updateTime: '2025-05-22 17:00', usage: 42, type: 'agriculture' },
    { id: 4, systemName: '应急指挥系统', owner: '张丽敏', status: '运行中', updateTime: '2025-05-22 16:00', usage: 30, type: 'emergency' }
  ])
  
  // 告警列表
  const alarmList = ref([
    { type: 'warning', time: '2025-05-01 10:30', content: '存储空间使用率超过70%' },
    { type: 'info', time: '2025-05-01 09:15', content: '系统例行维护完成' },
    { type: 'success', time: '2025-05-01 08:00', content: '数据备份任务完成' }
  ])
  
  // 加载状态
  const tableLoading = ref(false)
  
  // 刷新方法
  const refreshSystemStatus = () => {
    tableLoading.value = true
    setTimeout(() => {
      tableLoading.value = false
    }, 1000)
  }
  
  const refreshAlarms = () => {
    // 实现刷新逻辑
  }
  
  const trendChartRef = ref<HTMLElement | null>(null)
  let trendChart: echarts.ECharts | null = null
  
  const getTrendData = () => {
    // 根据 timeRange 返回不同的模拟数据
    if (timeRange.value === 'day') {
      return {
        x: Array.from({ length: 24 }, (_, i) => `${i}:00`),
        cpu: [65, 60, 62, 66, 68, 72, 69, 70, 68, 67, 66, 65, 64, 63, 62, 61, 60, 62, 64, 66, 68, 70, 72, 74],
        mem: [78, 75, 76, 77, 79, 81, 78, 77, 76, 75, 74, 73, 72, 71, 70, 72, 74, 76, 78, 80, 82, 84, 86, 88],
        storage: [45, 44, 44, 45, 45, 46, 46, 47, 47, 48, 48, 49, 49, 50, 50, 51, 51, 52, 52, 53, 53, 54, 54, 55]
      }
    } else if (timeRange.value === 'week') {
      return {
        x: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
        cpu: [65, 60, 70, 66, 68, 72, 69],
        mem: [78, 75, 80, 77, 79, 81, 78],
        storage: [45, 44, 46, 45, 47, 48, 45]
      }
    } else {
      return {
        x: Array.from({ length: 30 }, (_, i) => `${i + 1}日`),
        cpu: Array.from({ length: 30 }, () => Math.floor(60 + Math.random() * 15)),
        mem: Array.from({ length: 30 }, () => Math.floor(70 + Math.random() * 20)),
        storage: Array.from({ length: 30 }, () => Math.floor(40 + Math.random() * 15))
      }
    }
  }
  
  const renderTrendChart = () => {
    nextTick(() => {
      if (!trendChartRef.value) return
      if (trendChart) {
        trendChart.dispose()
      }
      trendChart = echarts.init(trendChartRef.value)
      const data = getTrendData()
      const option = {
        tooltip: { 
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#6a7985'
            }
          }
        },
        legend: { 
          data: ['CPU', '内存', '存储'],
          right: 10,
          top: 0
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: { 
          type: 'category', 
          data: data.x,
          boundaryGap: false
        },
        yAxis: { 
          type: 'value', 
          axisLabel: { formatter: '{value}%' } 
        },
        series: [
          { 
            name: 'CPU', 
            type: 'line', 
            data: data.cpu, 
            smooth: true,
            lineStyle: {
              width: 3
            },
            itemStyle: {
              borderWidth: 2
            },
            areaStyle: {
              opacity: 0.1
            }
          },
          { 
            name: '内存', 
            type: 'line', 
            data: data.mem, 
            smooth: true,
            lineStyle: {
              width: 3
            },
            itemStyle: {
              borderWidth: 2
            },
            areaStyle: {
              opacity: 0.1
            }
          },
          { 
            name: '存储', 
            type: 'line', 
            data: data.storage, 
            smooth: true,
            lineStyle: {
              width: 3
            },
            itemStyle: {
              borderWidth: 2
            },
            areaStyle: {
              opacity: 0.1
            }
          }
        ]
      }
      trendChart.setOption(option)
    })
  }
  
  onMounted(() => {
    renderTrendChart()
    window.addEventListener('resize', () => trendChart?.resize())
  })
  onBeforeUnmount(() => {
    trendChart?.dispose()
    window.removeEventListener('resize', () => trendChart?.resize())
  })
  watch(timeRange, () => {
    renderTrendChart()
  })

  // 添加业务系统相关函数和变量
  const getStatusType = (status: string) => {
    switch (status) {
      case '运行中': return 'success'
      case '维护中': return 'info'
      case '告警': return 'warning'
      case '离线': return 'danger'
      default: return 'info'
    }
  }

  const getSystemIcon = (row: any) => {
    switch (row.type) {
      case 'public': return 'Tickets'
      case 'village': return 'HomeFilled'
      case 'agriculture': return 'Crop'
      case 'emergency': return 'FirstAidKit'
      default: return 'Document'
    }
  }

  const getSystemIconClass = (row: any) => {
    switch (row.type) {
      case 'public': return 'icon-public'
      case 'village': return 'icon-village'
      case 'agriculture': return 'icon-agriculture'
      case 'emergency': return 'icon-emergency'
      default: return ''
    }
  }

  // 格式化时间函数
  const formatTime = (date: Date) => {
    return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')} ${String(date.getHours()).padStart(2, '0')}:${String(date.getMinutes()).padStart(2, '0')}`
  }

  // 添加操作函数
  const viewSystemDetail = (row: any) => {
    console.log('查看系统详情', row)
    // 实现查看详情逻辑
  }

  const configSystem = (row: any) => {
    console.log('配置系统', row)
    // 实现配置系统逻辑
  }

  const restartSystem = (row: any) => {
    console.log('重启系统', row)
    // 实现重启系统逻辑
  }
  </script>
  
  <style scoped>
  .dashboard-container {
    padding: 20px;
    background-color: #f5f7fa;
    min-height: calc(100vh - 60px);
  }
  
  .dashboard-header {
    margin-bottom: 24px;
    background: linear-gradient(to right, #1976d2, #2196f3);
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }
  
  .dashboard-title {
    font-size: 24px;
    margin: 0;
    color: white;
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
  }
  
  .stat-row {
    margin-bottom: 20px;
  }
  
  .stat-card {
    margin-bottom: 20px;
    border-radius: 8px;
    transition: all 0.3s;
    overflow: hidden;
  }
  
  .stat-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  }
  
  .stat-card.primary .stat-icon {
    color: #409EFF;
  }
  
  .stat-card.success .stat-icon {
    color: #67C23A;
  }
  
  .stat-card.warning .stat-icon {
    color: #E6A23C;
  }
  
  .stat-card.danger .stat-icon {
    color: #F56C6C;
  }
  
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .header-left {
    display: flex;
    align-items: center;
  }
  
  .header-left .el-icon {
    margin-right: 8px;
    font-size: 18px;
  }
  
  .stat-content {
    display: flex;
    align-items: center;
  }
  
  .stat-icon {
    font-size: 48px;
    margin-right: 16px;
  }
  
  .stat-info {
    flex: 1;
  }
  
  .stat-number {
    font-size: 28px;
    font-weight: bold;
    line-height: 1.2;
    color: #303133;
  }
  
  .stat-label {
    font-size: 14px;
    color: #909399;
    margin-top: 4px;
  }
  
  .system-status {
    margin-bottom: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.05);
  }
  
  .monitor-row {
    margin-bottom: 20px;
  }
  
  .monitor-card {
    margin-bottom: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.05);
  }
  
  .chart-container {
    height: 350px;
  }
  
  .alarm-content {
    padding: 8px;
    background-color: #f8f8f8;
    border-radius: 4px;
    margin-bottom: 8px;
  }
  
  :deep(.el-card__header) {
    padding: 15px 20px;
    border-bottom: 1px solid #ebeef5;
  }
  
  :deep(.el-card__body) {
    padding: 20px;
  }
  
  :deep(.el-timeline-item__node) {
    box-shadow: 0 0 0 2px #fff;
  }
  
  :deep(.el-table th) {
    background-color: #f5f7fa !important;
  }
  
  .system-summary {
    margin-bottom: 20px;
  }
  
  .summary-card {
    display: flex;
    align-items: center;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 15px;
    transition: all 0.3s;
  }
  
  .summary-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }
  
  .summary-card.running {
    background-color: rgba(103, 194, 58, 0.1);
    border: 1px solid rgba(103, 194, 58, 0.2);
  }
  
  .summary-card.maintenance {
    background-color: rgba(144, 147, 153, 0.1);
    border: 1px solid rgba(144, 147, 153, 0.2);
  }
  
  .summary-card.warning {
    background-color: rgba(230, 162, 60, 0.1);
    border: 1px solid rgba(230, 162, 60, 0.2);
  }
  
  .summary-card.offline {
    background-color: rgba(245, 108, 108, 0.1);
    border: 1px solid rgba(245, 108, 108, 0.2);
  }
  
  .summary-icon {
    width: 50px;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 24px;
    border-radius: 50%;
    margin-right: 15px;
  }
  
  .summary-card.running .summary-icon {
    background-color: #67C23A;
    color: white;
  }
  
  .summary-card.maintenance .summary-icon {
    background-color: #909399;
    color: white;
  }
  
  .summary-card.warning .summary-icon {
    background-color: #E6A23C;
    color: white;
  }
  
  .summary-card.offline .summary-icon {
    background-color: #F56C6C;
    color: white;
  }
  
  .summary-info {
    flex: 1;
  }
  
  .summary-title {
    font-size: 14px;
    color: #606266;
    margin-bottom: 5px;
  }
  
  .summary-value {
    font-size: 28px;
    font-weight: bold;
    color: #303133;
  }
  
  .system-name {
    display: flex;
    align-items: center;
  }
  
  .system-icon {
    margin-right: 8px;
    font-size: 18px;
  }
  
  .icon-public {
    color: #409EFF;
  }
  
  .icon-village {
    color: #67C23A;
  }
  
  .icon-agriculture {
    color: #E6A23C;
  }
  
  .icon-emergency {
    color: #F56C6C;
  }
  
  .header-actions {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .sync-time {
    font-size: 13px;
    color: #909399;
  }
  
  .ml-10 {
    margin-left: 10px;
  }
  
  @media (max-width: 768px) {
    .dashboard-container {
      padding: 10px;
    }
    
    .dashboard-header {
      padding: 15px;
    }
    
    .stat-icon {
      font-size: 36px;
      margin-right: 12px;
    }
    
    .stat-number {
      font-size: 22px;
    }
    
    .chart-container {
      height: 280px;
    }
  }
  </style>
