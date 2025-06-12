<template>
  <div class="emergency-container">
    <!-- 应急概览 -->
    <el-row :gutter="20" class="emergency-overview">
      <el-col :span="6">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>当前预警</span>
              <el-tag size="small" type="warning">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">3</div>
            <div class="unit">个</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>应急事件</span>
              <el-tag size="small" type="danger">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">2</div>
            <div class="unit">起</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>待调度资源</span>
              <el-tag size="small" type="warning">实时</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">8</div>
            <div class="unit">项</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="hover">
          <template #header>
            <div class="card-header">
              <span>本月事件处理率</span>
              <el-tag size="small" type="success">月度</el-tag>
            </div>
          </template>
          <div class="card-content">
            <div class="value">100</div>
            <div class="unit">%</div>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <!-- 应急管理标签页 -->
    <el-card class="emergency-management" shadow="hover">
      <el-tabs v-model="activeTab">
        <!-- 预警信息 -->
        <el-tab-pane label="预警信息" name="warning">
          <div class="tab-header">
            <el-form :inline="true" :model="warningSearchForm" class="search-form">
              <el-form-item>
                <el-input v-model="warningSearchForm.keyword" placeholder="预警标题" clearable />
              </el-form-item>
              <el-form-item>
                <el-select v-model="warningSearchForm.level" placeholder="预警等级" clearable>
                  <el-option label="红色预警" value="red" />
                  <el-option label="橙色预警" value="orange" />
                  <el-option label="黄色预警" value="yellow" />
                  <el-option label="蓝色预警" value="blue" />
                </el-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="handleWarningSearch">搜索</el-button>
                <el-button @click="resetWarningSearch">重置</el-button>
              </el-form-item>
            </el-form>
            <el-button type="primary" @click="handleAddWarning">发布预警</el-button>
          </div>
          <el-table :data="warningList" style="width: 100%">
            <el-table-column prop="title" label="预警标题" min-width="200" />
            <el-table-column prop="type" label="预警类型" width="120" />
            <el-table-column prop="level" label="预警等级" width="100">
              <template #default="{ row }">
                <el-tag :type="getWarningLevelType(row.level)">
                  {{ row.level }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="area" label="影响区域" width="150" />
            <el-table-column prop="publishTime" label="发布时间" width="180" />
            <el-table-column prop="status" label="状态" width="100">
              <template #default="{ row }">
                <el-tag :type="row.status === '已解除' ? 'success' : 'danger'">
                  {{ row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column label="操作" width="120">
              <template #default="{ row }">
                <el-dropdown>
                  <el-button link type="primary">
                    操作<el-icon class="el-icon--right"><arrow-down /></el-icon>
                  </el-button>
                  <template #dropdown>
                    <el-dropdown-menu>
                      <el-dropdown-item @click="viewWarning(row)">查看</el-dropdown-item>
                      <el-dropdown-item @click="handleWarning(row)">处理</el-dropdown-item>
                      <el-dropdown-item @click="deleteWarning(row)" divided>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </template>
                </el-dropdown>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 应急事件 -->
        <el-tab-pane label="应急事件" name="event">
          <div class="tab-header">
            <el-form :inline="true" :model="eventSearchForm" class="search-form">
              <el-form-item>
                <el-input v-model="eventSearchForm.keyword" placeholder="事件标题" clearable />
              </el-form-item>
              <el-form-item>
                <el-select v-model="eventSearchForm.status" placeholder="处理状态" clearable>
                  <el-option label="待处理" value="pending" />
                  <el-option label="处理中" value="processing" />
                  <el-option label="已完成" value="completed" />
                </el-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="handleEventSearch">搜索</el-button>
                <el-button @click="resetEventSearch">重置</el-button>
              </el-form-item>
            </el-form>
            <el-button type="primary" @click="handleAddEvent">新增事件</el-button>
          </div>
          <el-table :data="eventList" style="width: 100%">
            <el-table-column prop="title" label="事件标题" min-width="200" />
            <el-table-column prop="type" label="事件类型" width="120" />
            <el-table-column prop="level" label="事件等级" width="100">
              <template #default="{ row }">
                <el-tag :type="getEventLevelType(row.level)">
                  {{ row.level }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="location" label="事发地点" width="150" />
            <el-table-column prop="reportTime" label="上报时间" width="180" />
            <el-table-column prop="status" label="状态" width="100">
              <template #default="{ row }">
                <el-tag :type="getEventStatusType(row.status)">
                  {{ row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column label="操作" width="120">
              <template #default="{ row }">
                <el-dropdown>
                  <el-button link type="primary">
                    操作<el-icon class="el-icon--right"><arrow-down /></el-icon>
                  </el-button>
                  <template #dropdown>
                    <el-dropdown-menu>
                      <el-dropdown-item @click="viewEvent(row)">查看</el-dropdown-item>
                      <el-dropdown-item @click="handleEvent(row)">处理</el-dropdown-item>
                      <el-dropdown-item @click="deleteEvent(row)" divided>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </template>
                </el-dropdown>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 资源调度 -->
        <el-tab-pane label="资源调度" name="resource">
          <div class="tab-header">
            <el-form :inline="true" :model="resourceSearchForm" class="search-form">
              <el-form-item>
                <el-input v-model="resourceSearchForm.keyword" placeholder="资源名称" clearable />
              </el-form-item>
              <el-form-item>
                <el-select v-model="resourceSearchForm.type" placeholder="资源类型" clearable>
                  <el-option label="人员" value="personnel" />
                  <el-option label="物资" value="material" />
                  <el-option label="车辆" value="vehicle" />
                  <el-option label="设备" value="equipment" />
                </el-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="handleResourceSearch">搜索</el-button>
                <el-button @click="resetResourceSearch">重置</el-button>
              </el-form-item>
            </el-form>
            <el-button type="primary" @click="handleAddResource">新增资源</el-button>
          </div>
          <el-table :data="resourceList" style="width: 100%">
            <el-table-column prop="name" label="资源名称" min-width="150" />
            <el-table-column prop="type" label="资源类型" width="100" />
            <el-table-column prop="quantity" label="数量" width="100" />
            <el-table-column prop="location" label="存放位置" width="150" />
            <el-table-column prop="status" label="状态" width="100">
              <template #default="{ row }">
                <el-tag :type="row.status === '可用' ? 'success' : 'info'">
                  {{ row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="updateTime" label="更新时间" width="180" />
            <el-table-column label="操作" width="120">
              <template #default="{ row }">
                <el-dropdown>
                  <el-button link type="primary">
                    操作<el-icon class="el-icon--right"><arrow-down /></el-icon>
                  </el-button>
                  <template #dropdown>
                    <el-dropdown-menu>
                      <el-dropdown-item @click="viewResource(row)">查看</el-dropdown-item>
                      <el-dropdown-item @click="dispatchResource(row)">调度</el-dropdown-item>
                      <el-dropdown-item @click="deleteResource(row)" divided>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </template>
                </el-dropdown>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <!-- 应急预案 -->
        <el-tab-pane label="应急预案" name="plan">
          <div class="tab-header">
            <el-form :inline="true" :model="planSearchForm" class="search-form">
              <el-form-item>
                <el-input v-model="planSearchForm.keyword" placeholder="预案名称" clearable />
              </el-form-item>
              <el-form-item>
                <el-select v-model="planSearchForm.type" placeholder="预案类型" clearable>
                  <el-option label="自然灾害" value="natural" />
                  <el-option label="事故灾难" value="accident" />
                  <el-option label="公共卫生" value="health" />
                  <el-option label="社会安全" value="security" />
                </el-select>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="handlePlanSearch">搜索</el-button>
                <el-button @click="resetPlanSearch">重置</el-button>
              </el-form-item>
            </el-form>
            <el-button type="primary" @click="handleAddPlan">新增预案</el-button>
          </div>
          <el-table :data="planList" style="width: 100%">
            <el-table-column prop="name" label="预案名称" min-width="200" />
            <el-table-column prop="type" label="预案类型" width="120" />
            <el-table-column prop="level" label="响应等级" width="100">
              <template #default="{ row }">
                <el-tag :type="getPlanLevelType(row.level)">
                  {{ row.level }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="department" label="责任部门" width="150" />
            <el-table-column prop="updateTime" label="更新时间" width="180" />
            <el-table-column prop="status" label="状态" width="100">
              <template #default="{ row }">
                <el-tag :type="row.status === '已启用' ? 'success' : 'info'">
                  {{ row.status }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column label="操作" width="120">
              <template #default="{ row }">
                <el-dropdown>
                  <el-button link type="primary">
                    操作<el-icon class="el-icon--right"><arrow-down /></el-icon>
                  </el-button>
                  <template #dropdown>
                    <el-dropdown-menu>
                      <el-dropdown-item @click="viewPlan(row)">查看</el-dropdown-item>
                      <el-dropdown-item @click="editPlan(row)">编辑</el-dropdown-item>
                      <el-dropdown-item @click="deletePlan(row)" divided>删除</el-dropdown-item>
                    </el-dropdown-menu>
                  </template>
                </el-dropdown>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { ArrowDown } from '@element-plus/icons-vue'

// 当前激活的标签页
const activeTab = ref('warning')

// 预警搜索表单
const warningSearchForm = reactive({
  keyword: '',
  level: ''
})

// 事件搜索表单
const eventSearchForm = reactive({
  keyword: '',
  status: ''
})

// 资源搜索表单
const resourceSearchForm = reactive({
  keyword: '',
  type: ''
})

// 预案搜索表单
const planSearchForm = reactive({
  keyword: '',
  type: ''
})

// 预警列表数据
const warningList = ref([
  {
    id: 1,
    title: '暴雨橙色预警',
    type: '气象预警',
    level: '橙色预警',
    area: '张庄镇全域',
    publishTime: '2025-05-15 08:00',
    status: '生效中'
  },
  {
    id: 2,
    title: '大风黄色预警',
    type: '气象预警',
    level: '黄色预警',
    area: '张庄镇北部',
    publishTime: '2025-05-15 10:30',
    status: '已解除'
  }
])

// 事件列表数据
const eventList = ref([
  {
    id: 1,
    title: '道路积水严重',
    type: '自然灾害',
    level: '一般',
    location: '张庄村主干道',
    reportTime: '2025-05-15 09:15',
    status: '处理中'
  },
  {
    id: 2,
    title: '树木倒伏',
    type: '自然灾害',
    level: '一般',
    location: '李庄村道路',
    reportTime: '2025-05-15 11:20',
    status: '待处理'
  }
])

// 资源列表数据
const resourceList = ref([
  {
    id: 1,
    name: '应急抢险队',
    type: '人员',
    quantity: '20人',
    location: '镇政府',
    status: '可用',
    updateTime: '2025-05-15 08:00'
  },
  {
    id: 2,
    name: '抽水泵',
    type: '设备',
    quantity: '5台',
    location: '物资仓库',
    status: '可用',
    updateTime: '2025-05-15 08:00'
  }
])

// 预案列表数据
const planList = ref([
  {
    id: 1,
    name: '防汛抗旱应急预案',
    type: '自然灾害',
    level: '一级',
    department: '应急管理办公室',
    updateTime: '2025-05-01',
    status: '已启用'
  },
  {
    id: 2,
    name: '突发公共卫生事件应急预案',
    type: '公共卫生',
    level: '二级',
    department: '卫生健康办公室',
    updateTime: '2025-02-15',
    status: '已启用'
  }
])

// 获取预警等级对应的标签类型
const getWarningLevelType = (level: string) => {
  switch (level) {
    case '红色预警':
      return 'danger'
    case '橙色预警':
      return 'warning'
    case '黄色预警':
      return 'warning'
    case '蓝色预警':
      return 'info'
    default:
      return 'info'
  }
}

// 获取事件等级对应的标签类型
const getEventLevelType = (level: string) => {
  switch (level) {
    case '特别重大':
      return 'danger'
    case '重大':
      return 'warning'
    case '较大':
      return 'warning'
    case '一般':
      return 'info'
    default:
      return 'info'
  }
}

// 获取事件状态对应的标签类型
const getEventStatusType = (status: string) => {
  switch (status) {
    case '待处理':
      return 'warning'
    case '处理中':
      return 'primary'
    case '已完成':
      return 'success'
    default:
      return 'info'
  }
}

// 获取预案等级对应的标签类型
const getPlanLevelType = (level: string) => {
  switch (level) {
    case '一级':
      return 'danger'
    case '二级':
      return 'warning'
    case '三级':
      return 'info'
    case '四级':
      return 'info'
    default:
      return 'info'
  }
}

// 预警相关方法
const handleWarningSearch = () => {
  console.log('搜索条件:', warningSearchForm)
}

const resetWarningSearch = () => {
  warningSearchForm.keyword = ''
  warningSearchForm.level = ''
}

const handleAddWarning = () => {
  console.log('发布预警')
}

const viewWarning = (row: any) => {
  console.log('查看预警:', row)
}

const handleWarning = (row: any) => {
  console.log('处理预警:', row)
}

const deleteWarning = (row: any) => {
  console.log('删除预警:', row)
}

// 事件相关方法
const handleEventSearch = () => {
  console.log('搜索条件:', eventSearchForm)
}

const resetEventSearch = () => {
  eventSearchForm.keyword = ''
  eventSearchForm.status = ''
}

const handleAddEvent = () => {
  console.log('新增事件')
}

const viewEvent = (row: any) => {
  console.log('查看事件:', row)
}

const handleEvent = (row: any) => {
  console.log('处理事件:', row)
}

const deleteEvent = (row: any) => {
  console.log('删除事件:', row)
}

// 资源相关方法
const handleResourceSearch = () => {
  console.log('搜索条件:', resourceSearchForm)
}

const resetResourceSearch = () => {
  resourceSearchForm.keyword = ''
  resourceSearchForm.type = ''
}

const handleAddResource = () => {
  console.log('新增资源')
}

const viewResource = (row: any) => {
  console.log('查看资源:', row)
}

const dispatchResource = (row: any) => {
  console.log('调度资源:', row)
}

const deleteResource = (row: any) => {
  console.log('删除资源:', row)
}

// 预案相关方法
const handlePlanSearch = () => {
  console.log('搜索条件:', planSearchForm)
}

const resetPlanSearch = () => {
  planSearchForm.keyword = ''
  planSearchForm.type = ''
}

const handleAddPlan = () => {
  console.log('新增预案')
}

const viewPlan = (row: any) => {
  console.log('查看预案:', row)
}

const editPlan = (row: any) => {
  console.log('编辑预案:', row)
}

const deletePlan = (row: any) => {
  console.log('删除预案:', row)
}
</script>

<style scoped>
.emergency-container {
  padding: 20px;
}

.emergency-overview {
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

.emergency-management {
  margin-bottom: 20px;
}

.tab-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.search-form {
  display: flex;
  align-items: center;
}

:deep(.el-card__header) {
  padding: 15px 20px;
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
</style> 