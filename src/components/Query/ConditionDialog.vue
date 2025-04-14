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
          <el-button size="small" type="danger">删除</el-button>
          <el-button size="small">新建</el-button>
        </div>
      </div>

          <div class="filter-dialog__right">
        <el-tabs v-model="activeTab" class="filter-dialog__tabs">
          <el-tab-pane label="条件" name="condition">
            <div class="tab-actions">
              <el-button class="mt-2" size="small" type="primary" @click="addCondition">添加条件</el-button>
            </div>
            <el-table :data="conditions" border style="width: 100%">
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
              <el-button class="mt-2" size="small" type="primary" @click="addColumn">添加栏目</el-button>
            </div>
            <el-table :data="columns" border style="width: 100%">
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
              <el-table-column label="操作" width="80">
                <template #default="{ $index }">
                  <el-button link type="danger" icon="Delete" @click="removeColumn($index)" />
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="排序" name="sort">
            <div class="tab-actions">
              <el-button class="mt-2" size="small" type="primary" @click="addCondition">添加排序</el-button>
            </div>
            <el-table :data="conditions" border style="width: 100%">
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

<script setup lang="ts">
import { ref, reactive, watch } from 'vue'

const props = defineProps<{ modelValue: boolean }>()
const emit = defineEmits(['update:modelValue', 'confirm'])

const dialogVisible = ref(props.modelValue)

watch(
    () => props.modelValue,
    (val) => {
      dialogVisible.value = val
    }
)

watch(dialogVisible, (val) => emit('update:modelValue', val))

const treeData = ref([
  { id: 1, label: '模板一' },
  { id: 2, label: '模板二' },
])

const defaultProps = {
  children: 'children',
  label: 'label',
}

const handleTreeClick = (node: any) => {
  console.log('选择模板:', node)
}

const activeTab = ref('condition')

interface Condition {
  field: string
  label: string
  queryType: string
  defaultValue: string
}

interface Column {
  field: string
  label: string
  width: number
  fixed: boolean
  summary: boolean
  type: string
}

const conditions = ref<Condition[]>([])
const columns = ref<Column[]>([])

const queryTypes = [
  { label: '模糊', value: 'like' },
  { label: '等于', value: 'equal' },
  { label: '日期区间', value: 'dateRange' },
]

const addCondition = () => {
  conditions.value.push({ field: '字段名', label: '', queryType: '', defaultValue: '' })
}

const removeCondition = (index: number) => {
  conditions.value.splice(index, 1)
}

const addColumn = () => {
  columns.value.push({ field: '字段名', label: '', width: 120, fixed: false, summary: false, type: 'string' })
}

const removeColumn = (index: number) => {
  columns.value.splice(index, 1)
}

const templateName = ref('')

const handleConfirm = () => {
  emit('confirm', {
    name: templateName.value,
    conditions: conditions.value,
    columns: columns.value,
  })
  dialogVisible.value = false
}

const handleCancel = () => {
  dialogVisible.value = false
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
      margin-top: auto; /* 将按钮推到底部 */
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
  }

  &__tabs {
    flex: 1;
    overflow: auto;

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
  justify-content: flex-end;
  margin: 0px 0; /* 减小上下边距 */

  :deep(.el-button) {
    padding: 6px 12px; /* 调整按钮内边距使其更小 */
    font-size: 12px; /* 可选：减小字体大小 */
  }
}
</style>
