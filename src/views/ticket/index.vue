<template>
  <div>
    <el-card>
      <template #header>
        <div class="flex justify-between items-center">
          <el-button type="primary" icon="Setting" @click="handleOpenDialog">设置</el-button>
        </div>
      </template>

      <el-table :data="tableData" border v-loading="loading" style="width: 100%">
        <el-table-column label="工单编号" prop="ticketNo" width="150" />
        <el-table-column label="标题" prop="title" />
        <el-table-column label="状态" prop="status" width="100">
          <template #default="{ row }">
            {{ statusMap[row.status] || '未知' }}
          </template>
        </el-table-column>
        <el-table-column label="优先级" prop="priority" width="100">
          <template #default="{ row }">
            {{ priorityMap[row.priority] || '未知' }}
          </template>
        </el-table-column>
        <el-table-column label="创建人" prop="createBy" width="120" />
        <el-table-column label="创建时间" prop="createTime" width="180" />
        <el-table-column label="更新人" prop="updateBy" width="120" />
        <el-table-column label="更新时间" prop="updateTime" width="180" />
        <el-table-column label="备注" prop="remark" />
      </el-table>
    </el-card>

    <!-- 条件设置对话框 -->
    <ConditionDialog
        v-model="dialogVisible"
        :conditionsData="conditions"
        :columnsData="columns"
        :sortData="sortFields"
        :templateData="templates"
        @confirm="handleConfirm"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import ConditionDialog from '@/components/Query/ConditionDialog.vue'

const dialogVisible = ref(false);
const loading = ref(false);
const tableData = ref([]);
const total = ref(0);
const originalData = ref([]);

const statusMap = {
  '0': '待处理',
  '1': '处理中',
  '2': '已完成'
};

const priorityMap = {
  '0': '低',
  '1': '中',
  '2': '高'
};

// 初始化查询条件
const conditions = ref([
  { field: 'ticketNo', label: '工单编号', queryType: 'equal', defaultValue: '' },
  { field: 'title', label: '标题', queryType: 'like', defaultValue: '异常' },
  { field: 'status', label: '状态', queryType: 'equal', defaultValue: '0' }
]);

// 初始条件设置
const initialConditions = ref(conditions.value);

// 添加列和排序数据
const columns = ref([
  { field: 'ticketNo', label: '工单编号', width: 150, fixed: false, summary: false, type: 'string' },
  { field: 'title', label: '标题', width: 200, fixed: false, summary: false, type: 'string' },
  { field: 'status', label: '状态', width: 100, fixed: false, summary: false, type: 'string' },
  { field: 'priority', label: '优先级', width: 100, fixed: false, summary: false, type: 'string' }
]);

const sortFields = ref([
  { field: 'createTime', label: '创建时间', queryType: 'desc', defaultValue: '' }
]);

const templates = ref([
  { id: 1, label: '默认模板' },
  { id: 2, label: '自定义模板' }
]);

// 修改打开对话框的方法
const handleOpenDialog = () => {
  // 先刷新数据
  getList();
  
  // 更新条件、列和排序数据
  updateDialogData();
  
  // 然后打开对话框
  dialogVisible.value = true;
}

// 添加更新对话框数据的方法
const updateDialogData = () => {
  // 这里可以从后端获取最新的条件、列和排序数据
  // 或者根据当前表格状态更新
  
  // 示例：更新条件数据
  conditions.value = [
    { field: 'ticketNo', label: '工单编号', queryType: 'equal', defaultValue: '' },
    { field: 'title', label: '标题', queryType: 'like', defaultValue: '' },
    { field: 'status', label: '状态', queryType: 'equal', defaultValue: '' },
    { field: 'priority', label: '优先级', queryType: 'equal', defaultValue: '' }
  ];
  
  // 示例：更新列数据
  columns.value = [
    { field: 'ticketNo', label: '工单编号', width: 150, fixed: false, summary: false, type: 'string' },
    { field: 'title', label: '标题', width: 200, fixed: false, summary: false, type: 'string' },
    { field: 'status', label: '状态', width: 100, fixed: false, summary: false, type: 'string' },
    { field: 'priority', label: '优先级', width: 100, fixed: false, summary: false, type: 'string' },
    { field: 'createBy', label: '创建人', width: 120, fixed: false, summary: false, type: 'string' },
    { field: 'createTime', label: '创建时间', width: 180, fixed: false, summary: false, type: 'date' }
  ];
  
  // 示例：更新排序数据
  sortFields.value = [
    { field: 'createTime', label: '创建时间', queryType: 'desc', defaultValue: '' }
  ];
}

const getList = () => {
  loading.value = true;
  setTimeout(() => {
    originalData.value = [
      {
        id: 1,
        ticketNo: 'TK20230001',
        title: '系统登录异常',
        status: '0',
        priority: '2',
        createBy: '张三',
        createTime: '2023-01-01 10:00:00',
        updateBy: '李四',
        updateTime: '2023-01-02 14:30:00',
        remark: '用户反馈无法登录系统'
      },
      {
        id: 2,
        ticketNo: 'TK20230002',
        title: '数据导出功能失效',
        status: '1',
        priority: '1',
        createBy: '王五',
        createTime: '2023-01-03 09:15:00',
        updateBy: '赵六',
        updateTime: '2023-01-03 11:20:00',
        remark: '导出Excel时报错'
      },
      {
        id: 3,
        ticketNo: 'TK20230003',
        title: '新增用户权限配置',
        status: '2',
        priority: '0',
        createBy: '张三',
        createTime: '2023-01-05 16:40:00',
        updateBy: '张三',
        updateTime: '2023-01-06 10:30:00',
        remark: '需要为新部门配置系统权限'
      }
    ];
    tableData.value = [...originalData.value];
    total.value = tableData.value.length;
    loading.value = false;
  }, 500);
};

// 处理查询条件确认
const handleConfirm = (data) => {
  console.log('查询条件：', data);

  // 根据条件筛选数据（简单示例）
  if (data && data.conditions && data.conditions.length > 0) {
    tableData.value = originalData.value.filter(item => {
      return data.conditions.every((condition) => {
        if (!condition.field || !condition.queryType) return true;

        const fieldValue = item[condition.field];
        const defaultValue = condition.defaultValue;

        if (!defaultValue) return true;

        switch (condition.queryType) {
          case 'like':
            return String(fieldValue).includes(defaultValue);
          case 'equal':
            return String(fieldValue) === defaultValue;
          default:
            return true;
        }
      });
    });
  } else {
    tableData.value = [...originalData.value];
  }
};

onMounted(() => {
  getList();
});
</script>
