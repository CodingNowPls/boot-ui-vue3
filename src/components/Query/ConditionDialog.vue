<template>
  <el-dialog v-model="dialogVisible" title="查询" width="1000px" class="filter-dialog">
    <div class="filter-dialog__body">
      <!-- 左侧目录树 -->
      <div class="filter-dialog__left">
        <el-tree
            :data="treeData"
            :props="defaultProps"
            node-key="id"
            highlight-current
            @node-click="handleTreeClick"
        />
        <!-- 左侧按钮区域 -->
        <div class="filter-dialog__left-buttons">
          <el-button size="small">另存</el-button>
          <el-button size="small">新建</el-button>
          <el-button size="small" type="danger">删除</el-button>
        </div>
      </div>

          <div class="filter-dialog__right">
        <el-tabs v-model="activeTab" class="filter-dialog__tabs">
          <el-tab-pane label="条件" name="condition">
            <div class="tab-actions">
              <div class="move-buttons">
                <el-button size="small" :disabled="!hasSelectedItem || isFirstItem" @click="moveToTop">
                  <el-icon><Top /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isFirstItem" @click="moveUp">
                  <el-icon><ArrowUp /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isLastItem" @click="moveDown">
                  <el-icon><ArrowDown /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isLastItem" @click="moveToBottom">
                  <el-icon><Bottom /></el-icon>
                </el-button>
              </div>
              <el-button class="mt-2" size="small" type="primary" @click="addCondition">添加条件</el-button>
            </div>
            <el-table
              :data="conditions"
              border
              style="width: 100%"
              @row-click="handleRowClick('condition', $event)"
              highlight-current-row
            >
              <el-table-column label="序号" type="index" width="60" />
              <el-table-column label="显示项" prop="field" />
              <el-table-column label="显示名称">
                <template #default="{ row }">
                  <el-input v-model="row.label" size="small" />
                </template>
              </el-table-column>
              <el-table-column label="查询方式">
                <template #default="{ row }">
                  <el-select v-model="row.queryType" size="small" placeholder="请选择">
                    <el-option v-for="item in queryTypes" :key="item.value" :label="item.label" :value="item.value" />
                  </el-select>
                </template>
              </el-table-column>
              <el-table-column label="默认值">
                <template #default="{ row }">
                  <el-input v-model="row.defaultValue" size="small" />
                </template>
              </el-table-column>
              <el-table-column label="操作" width="80">
                <template #default="{ $index }">
                  <el-button link type="danger" icon="el-icon-delete" @click="removeCondition($index)" />
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>

          <el-tab-pane label="栏目" name="column">
            <div class="tab-actions">
              <div class="move-buttons">
                <el-button size="small" :disabled="!hasSelectedItem || isFirstItem" @click="moveToTop">
                  <el-icon><Top /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isFirstItem" @click="moveUp">
                  <el-icon><ArrowUp /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isLastItem" @click="moveDown">
                  <el-icon><ArrowDown /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isLastItem" @click="moveToBottom">
                  <el-icon><Bottom /></el-icon>
                </el-button>
              </div>
              <el-button class="mt-2" size="small" type="primary" @click="addColumn">添加栏目</el-button>
            </div>
            <el-table
              :data="columns"
              border
              style="width: 100%"
              @row-click="handleRowClick('column', $event)"
              highlight-current-row
            >
              <el-table-column label="序号" type="index" width="60" />
              <el-table-column label="显示项" prop="field" />
              <el-table-column label="显示名称">
                <template #default="{ row }">
                  <el-input v-model="row.label" size="small" />
                </template>
              </el-table-column>
              <el-table-column label="表格列宽">
                <template #default="{ row }">
                  <el-input-number v-model="row.width" size="small" :min="50" :max="500" :step="10" />
                </template>
              </el-table-column>
              <el-table-column label="是否冻结">
                <template #default="{ row }">
                  <el-checkbox v-model="row.fixed" />
                </template>
              </el-table-column>
              <el-table-column label="是否合计">
                <template #default="{ row }">
                  <el-checkbox v-model="row.summary" v-if="row.type === 'number'" />
                  <span v-else>-</span>
                </template>
              </el-table-column>
              <el-table-column label="操作" width="180">
                <template #default="{ $index }">
                  <div class="operation-buttons">
                    <el-button link type="danger" size="small" @click="removeColumn($index)">
                      <el-icon><Delete /></el-icon>
                    </el-button>
                  </div>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="排序" name="sort">
            <div class="tab-actions">
              <div class="move-buttons">
                <el-button size="small" :disabled="!hasSelectedItem || isFirstItem" @click="moveToTop">
                  <el-icon><Top /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isFirstItem" @click="moveUp">
                  <el-icon><ArrowUp /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isLastItem" @click="moveDown">
                  <el-icon><ArrowDown /></el-icon>
                </el-button>
                <el-button size="small" :disabled="!hasSelectedItem || isLastItem" @click="moveToBottom">
                  <el-icon><Bottom /></el-icon>
                </el-button>
              </div>
              <el-button class="mt-2" size="small" type="primary" @click="addSort">添加排序</el-button>
            </div>
            <el-table
              :data="sortFields"
              border
              style="width: 100%"
              @row-click="handleRowClick('sort', $event)"
              highlight-current-row
            >
              <el-table-column label="序号" type="index" width="60" />
              <el-table-column label="显示项" prop="field" />
              <el-table-column label="显示名称">
                <template #default="{ row }">
                  <el-input v-model="row.label" size="small" />
                </template>
              </el-table-column>
              <el-table-column label="排序方向">
                <template #default="{ row }">
                  <el-select v-model="row.queryType" size="small" placeholder="请选择">
                    <el-option label="升序" value="asc" />
                    <el-option label="降序" value="desc" />
                  </el-select>
                </template>
              </el-table-column>
              <el-table-column label="操作" width="80">
                <template #default="{ $index }">
                  <el-button link type="danger" icon="el-icon-delete" @click="removeSort($index)" />
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="高级" name="advanced"></el-tab-pane>
        </el-tabs>

        <!-- 右侧名称区域 -->
        <div class="filter-dialog__name-area">
          <el-form inline class="center-form">
            <el-form-item label="名称">
              <el-input v-model="templateName" placeholder="请输入名称" size="small" />
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>

    <!-- 底部操作栏 -->
    <template #footer>
      <div class="filter-dialog__footer">
        <div></div> <!-- 占位元素保持布局 -->
        <div class="filter-dialog__footer-buttons">
          <el-button size="small" @click="handleCancel">取消</el-button>
          <el-button size="small" type="primary" @click="handleConfirm">确定</el-button>
        </div>
      </div>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref, watch, computed, onMounted } from 'vue'
import { Top, ArrowUp, ArrowDown, Bottom, Delete } from '@element-plus/icons-vue'

const props = defineProps({
  modelValue: Boolean,
  conditionsData: {
    type: Array,
    default: () => []
  },
  columnsData: {
    type: Array,
    default: () => []
  },
  sortData: {
    type: Array,
    default: () => []
  },
  templateData: {
    type: Array,
    default: () => []
  }
})
const emit = defineEmits(['update:modelValue', 'confirm'])

const dialogVisible = ref(props.modelValue)

watch(
    () => props.modelValue,
    (val) => {
      dialogVisible.value = val
      if (val) {
        refreshData()
      }
    }
)

watch(dialogVisible, (val) => emit('update:modelValue', val))

watch(
  () => props.conditionsData,
  (val) => {
    if (val && val.length > 0) {
      conditions.value = JSON.parse(JSON.stringify(val))
    }
  }
)

watch(
  () => props.columnsData,
  (val) => {
    if (val && val.length > 0) {
      columns.value = JSON.parse(JSON.stringify(val))
    }
  }
)

watch(
  () => props.sortData,
  (val) => {
    if (val && val.length > 0) {
      sortFields.value = JSON.parse(JSON.stringify(val))
    }
  }
)

watch(
  () => props.templateData,
  (val) => {
    if (val && val.length > 0) {
      treeData.value = JSON.parse(JSON.stringify(val))
    }
  }
)

const refreshData = () => {
  if (props.conditionsData && props.conditionsData.length > 0) {
    conditions.value = JSON.parse(JSON.stringify(props.conditionsData))
  }

  if (props.columnsData && props.columnsData.length > 0) {
    columns.value = JSON.parse(JSON.stringify(props.columnsData))
  }

  if (props.sortData && props.sortData.length > 0) {
    sortFields.value = JSON.parse(JSON.stringify(props.sortData))
  }

  if (props.templateData && props.templateData.length > 0) {
    treeData.value = JSON.parse(JSON.stringify(props.templateData))
  }

  selectedTab.value = activeTab.value
  selectedIndex.value = -1
}

const treeData = ref([
  { id: 1, label: '模板一' },
  { id: 2, label: '模板二' },
])

const defaultProps = {
  children: 'children',
  label: 'label',
}

const handleTreeClick = (node) => {
  console.log('选择模板:', node)
}

const activeTab = ref('condition')

const conditions = ref([])
const columns = ref([])

const queryTypes = [
  { label: '模糊', value: 'like' },
  { label: '等于', value: 'equal' },
  { label: '日期区间', value: 'dateRange' },
]

const addCondition = () => {
  conditions.value.push({ field: '字段名', label: '', queryType: '', defaultValue: '' })
}

const removeCondition = (index) => {
  conditions.value.splice(index, 1)
}

const addColumn = () => {
  columns.value.push({ field: '字段名', label: '', width: 120, fixed: false, summary: false, type: 'string' })
}

const removeColumn = (index) => {
  columns.value.splice(index, 1)
}

const templateName = ref('')

const handleConfirm = () => {
  emit('confirm', {
    name: templateName.value,
    conditions: conditions.value,
    columns: columns.value,
    sortFields: sortFields.value
  })
  dialogVisible.value = false
}

const handleCancel = () => {
  dialogVisible.value = false
}

const sortFields = ref([])

const selectedTab = ref('')
const selectedIndex = ref(-1)
const hasSelectedItem = computed(() => selectedIndex.value >= 0)
const isFirstItem = computed(() => selectedIndex.value === 0)
const isLastItem = computed(() => {
  if (selectedIndex.value < 0) return true

  const currentData = getCurrentData()
  return selectedIndex.value === currentData.length - 1
})

const getCurrentData = () => {
  switch (selectedTab.value) {
    case 'condition': return conditions.value
    case 'column': return columns.value
    case 'sort': return sortFields.value
    default: return []
  }
}

const handleRowClick = (tab, row, rowIndex) => {
  selectedTab.value = tab
  selectedIndex.value = rowIndex !== undefined ? rowIndex : getCurrentData().indexOf(row)
}

const moveUp = () => {
  if (selectedIndex.value <= 0) return

  const data = getCurrentData()
  const temp = data[selectedIndex.value]
  data[selectedIndex.value] = data[selectedIndex.value - 1]
  data[selectedIndex.value - 1] = temp
  selectedIndex.value--
}

const moveDown = () => {
  const data = getCurrentData()
  if (selectedIndex.value < 0 || selectedIndex.value >= data.length - 1) return

  const temp = data[selectedIndex.value]
  data[selectedIndex.value] = data[selectedIndex.value + 1]
  data[selectedIndex.value + 1] = temp
  selectedIndex.value++
}

const moveToTop = () => {
  if (selectedIndex.value <= 0) return

  const data = getCurrentData()
  const item = data.splice(selectedIndex.value, 1)[0]
  data.unshift(item)
  selectedIndex.value = 0
}

const moveToBottom = () => {
  const data = getCurrentData()
  if (selectedIndex.value < 0 || selectedIndex.value >= data.length - 1) return

  const item = data.splice(selectedIndex.value, 1)[0]
  data.push(item)
  selectedIndex.value = data.length - 1
}

watch(activeTab, () => {
  selectedTab.value = activeTab.value
  selectedIndex.value = -1
})

const addSort = () => {
  sortFields.value.push({ field: '字段名', label: '', queryType: 'asc', defaultValue: '' })
}

const removeSort = (index) => {
  sortFields.value.splice(index, 1)
}
</script>

<style scoped lang="scss">
.filter-dialog {
  :deep(.el-dialog) {
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }

  &__body {
    display: flex;
    height: 420px;
    overflow: hidden;
  }

  &__left {
    width: 220px;
    margin-right: 20px;
    border-right: 1px solid #ebeef5;
    display: flex;
    flex-direction: column;
    padding-right: 10px;

    :deep(.el-tree) {
      margin-bottom: 10px;
      border-radius: 4px;
      padding: 5px;
      background-color: #fafafa;
    }

    &-buttons {
      margin-top: auto;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      gap: 10px;
      background-color: #f8f9fa;
      padding: 12px 8px;
      border-radius: 4px;
      box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
    }
  }

  &__right {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  &__tabs {
    flex: 1;
    overflow: auto;
    position: relative;

    :deep(.el-tabs__content) {
      overflow: visible;
      height: calc(100% - 40px);
    }

    :deep(.el-tab-pane) {
      height: 100%;
      overflow: auto;
    }

    :deep(.el-table) {
      margin-top: 2px;
      border-radius: 4px;
      overflow: hidden;
    }

    :deep(.el-button) {
      margin-bottom: 0px;
    }
  }

  &__name-area {
    margin-top: auto;
    padding-top: 16px;
    border-top: 1px solid #ebeef5;

    .center-form {
      display: flex;
      justify-content: center;
      width: 100%;

      :deep(.el-input) {
        width: 220px;
      }
    }
  }

  &__footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-top: 10px;

    &-buttons {
      display: flex;
      gap: 8px;
    }
  }
}

.tab-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0px 0;

  .move-buttons {
    display: flex;
    gap: 2px;

    :deep(.el-button) {
      padding: 4px;
      margin: 0;
      font-size: 12px;
    }
  }

  :deep(.el-button) {
    padding: 6px 12px;
    font-size: 12px;
  }
}

.operation-buttons {
  display: flex;
  gap: 2px;
  justify-content: center;

  :deep(.el-button) {
    padding: 4px;
    margin: 0;
  }
}

:deep(.el-dialog__body) {
  overflow: hidden;
}
</style>
