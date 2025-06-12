<template>
  <div class="public-service-container">
    <!-- 服务分类导航 -->
    <el-row :gutter="20" class="service-nav">
      <el-col :xs="24" :sm="12" :md="6" v-for="(category, index) in serviceCategories" :key="index">
        <el-card shadow="hover" class="category-card" @click="handleCategoryClick(category)">
          <div class="category-content">
            <el-icon class="category-icon" :size="40">
              <component :is="category.icon" />
            </el-icon>
            <div class="category-info">
              <h3>{{ category.title }}</h3>
              <p>{{ category.description }}</p>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 热门服务 -->
    <el-card class="hot-services" shadow="hover">
      <template #header>
        <div class="card-header">
          <span>热门服务</span>
          <el-button type="primary" link :icon="MoreFilled">更多服务</el-button>
        </div>
      </template>
      <el-row :gutter="20">
        <el-col :xs="12" :sm="8" :md="6" v-for="(service, index) in hotServices" :key="index">
          <div class="service-item" @click="handleServiceClick(service)">
            <el-icon :size="24">
              <component :is="service.icon" />
            </el-icon>
            <span>{{ service.name }}</span>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <!-- 办事指南 -->
    <el-card class="guide-section" shadow="hover">
      <template #header>
        <div class="card-header">
          <span>办事指南</span>
          <el-button type="primary" size="small" :icon="Plus" @click="handleAddGuide">发布指南</el-button>
        </div>
      </template>
      <el-table :data="guideList" style="width: 100%" v-loading="guideLoading" border stripe>
        <el-table-column prop="title" label="标题" min-width="200" />
        <el-table-column prop="department" label="发布部门" width="150" />
        <el-table-column prop="updateTime" label="更新时间" width="180" />
        <el-table-column prop="views" label="浏览量" width="100" />
        <el-table-column label="操作" width="140" fixed="right">
          <template #default="{ row }">
            <el-button-group>
              <el-button type="primary" :icon="View" circle @click="viewGuide(row)" title="查看" />
              <el-button type="warning" :icon="Edit" circle @click="editGuide(row)" title="编辑" />
              <el-button type="danger" :icon="Delete" circle @click="deleteGuide(row)" title="删除" />
            </el-button-group>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination-container">
        <el-pagination
          v-model:current-page="guidePage"
          v-model:page-size="guidePageSize"
          :page-sizes="[5, 10, 20, 50]"
          :total="guideTotal"
          layout="total, sizes, prev, pager, next, jumper"
        />
      </div>
    </el-card>

    <!-- 在线咨询 -->
    <el-card class="consultation-section" shadow="hover">
      <template #header>
        <div class="card-header">
          <span>在线咨询</span>
          <el-button type="primary" size="small" :icon="Plus" @click="handleNewConsultation">发起咨询</el-button>
        </div>
      </template>
      <el-table :data="consultationList" style="width: 100%" v-loading="consultationLoading" border stripe>
        <el-table-column prop="title" label="咨询主题" min-width="200" />
        <el-table-column prop="user" label="咨询人" width="120" />
        <el-table-column prop="department" label="回复部门" width="150" />
        <el-table-column prop="status" label="状态" width="100">
          <template #default="{ row }">
            <el-tag :type="row.status === '已回复' ? 'success' : 'warning'">
              {{ row.status }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="createTime" label="咨询时间" width="180" />
        <el-table-column label="操作" width="100" fixed="right">
          <template #default="{ row }">
            <el-button type="primary" :icon="View" circle @click="viewConsultation(row)" title="查看" />
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination-container">
        <el-pagination
          v-model:current-page="consultationPage"
          v-model:page-size="consultationPageSize"
          :page-sizes="[5, 10, 20, 50]"
          :total="consultationTotal"
          layout="total, sizes, prev, pager, next, jumper"
        />
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import {
  Document,
  House,
  School,
  FirstAidKit,
  User,
  Money,
  OfficeBuilding,
  Setting,
  ArrowDown,
  Plus,
  View,
  Edit,
  Delete,
  MoreFilled
} from '@element-plus/icons-vue'

// 服务分类
const serviceCategories = [
  {
    title: '政务服务',
    description: '提供各类政务事项办理服务',
    icon: 'Document'
  },
  {
    title: '便民服务',
    description: '提供日常生活便民服务',
    icon: 'House'
  },
  {
    title: '教育服务',
    description: '提供教育相关服务',
    icon: 'School'
  },
  {
    title: '医疗服务',
    description: '提供医疗健康服务',
    icon: 'FirstAidKit'
  }
]

// 热门服务
const hotServices = [
  { name: '社保查询', icon: 'User' },
  { name: '公积金查询', icon: 'Money' },
  { name: '户籍办理', icon: 'Document' },
  { name: '婚姻登记', icon: 'User' },
  { name: '生育服务', icon: 'FirstAidKit' },
  { name: '教育服务', icon: 'School' },
  { name: '就业服务', icon: 'OfficeBuilding' },
  { name: '养老服务', icon: 'User' }
]

// 办事指南列表
const guideList = ref([
  {
    id: 1,
    title: '城乡居民养老保险办理指南',
    department: '社保中心',
    updateTime: '2025-05-15 10:00',
    views: 1250
  },
  {
    id: 2,
    title: '新生儿落户办理流程',
    department: '派出所',
    updateTime: '2025-05-14 15:30',
    views: 980
  },
  {
    id: 3,
    title: '个体工商户注册登记指南',
    department: '市场监督管理局',
    updateTime: '2025-05-13 09:15',
    views: 1560
  }
])
const guideLoading = ref(false)
const guidePage = ref(1)
const guidePageSize = ref(10)
const guideTotal = ref(3)

// 在线咨询列表
const consultationList = ref([
  {
    id: 1,
    title: '关于社保卡办理的问题咨询',
    user: '张先生',
    department: '社保中心',
    status: '已回复',
    createTime: '2025-05-15 14:30'
  },
  {
    id: 2,
    title: '新生儿医保办理流程咨询',
    user: '李女士',
    department: '医保局',
    status: '待回复',
    createTime: '2025-05-15 16:45'
  }
])
const consultationLoading = ref(false)
const consultationPage = ref(1)
const consultationPageSize = ref(10)
const consultationTotal = ref(2)

// 处理分类点击
const handleCategoryClick = (category: any) => {
  console.log('点击分类:', category)
}

// 处理服务点击
const handleServiceClick = (service: any) => {
  console.log('点击服务:', service)
}

// 查看指南
const viewGuide = (row: any) => {
  console.log('查看指南:', row)
}

// 编辑指南
const editGuide = (row: any) => {
  console.log('编辑指南:', row)
}

// 删除指南
const deleteGuide = (row: any) => {
  console.log('删除指南:', row)
}

// 发布指南
const handleAddGuide = () => {
  console.log('发布新指南')
}

// 查看咨询
const viewConsultation = (row: any) => {
  console.log('查看咨询:', row)
}

// 发起咨询
const handleNewConsultation = () => {
  console.log('发起新咨询')
}
</script>

<style scoped>
.public-service-container {
  padding: 20px;
  background: #f5f7fa;
  min-height: calc(100vh - 60px);
}

.service-nav {
  margin-bottom: 20px;
}

.category-card {
  cursor: pointer;
  border-radius: 8px;
  transition: box-shadow 0.2s, transform 0.2s;
}
.category-card:hover {
  box-shadow: 0 2px 12px 0 rgba(64,158,255,0.12);
  transform: translateY(-3px);
}

.category-content {
  display: flex;
  align-items: center;
  padding: 20px;
}

.category-icon {
  margin-right: 15px;
  color: #409EFF;
}

.category-info h3 {
  margin: 0 0 5px 0;
  font-size: 18px;
  color: #303133;
}

.category-info p {
  margin: 0;
  font-size: 14px;
  color: #909399;
}

.hot-services {
  margin-bottom: 20px;
  border-radius: 8px;
}

.service-item {
  display: flex;
  align-items: center;
  padding: 15px;
  cursor: pointer;
  border-radius: 4px;
  transition: background 0.2s;
}
.service-item:hover {
  background: #ecf5ff;
}

.service-item .el-icon {
  margin-right: 10px;
  color: #409EFF;
}

.guide-section,
.consultation-section {
  margin-bottom: 20px;
  border-radius: 8px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.pagination-container {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}

:deep(.el-card__header) {
  padding: 15px 20px;
  border-bottom: 1px solid #ebeef5;
}

:deep(.el-card__body) {
  padding: 20px;
}

:deep(.el-button) {
  transition: none;
}
:deep(.el-button:hover) {
  opacity: 1;
}
:deep(.el-button--primary):not(.is-link) {
  background-color: #409EFF;
  border-color: #409EFF;
  color: #fff;
}
:deep(.el-button--primary:not(.is-link):hover) {
  background-color: #409EFF;
  border-color: #409EFF;
  color: #fff;
}
:deep(.el-button--primary.is-link),
:deep(.el-button--primary.is-link:hover) {
  background: none !important;
  border: none !important;
  color: #409EFF !important;
  box-shadow: none !important;
}
@media screen and (max-width: 768px) {
  .public-service-container {
    padding: 10px;
  }
  .category-content {
    padding: 10px;
  }
  .service-item {
    padding: 10px;
  }
}
</style> 