<template>
  <div class="layout-padding">
    <div class="layout-padding-auto layout-padding-view">
      <!-- 查询表单区域 -->
      <el-row v-show="showSearch">
        <el-form :model="state.queryForm" ref="queryRef" :inline="true" @keyup.enter="getDataList">
          <el-form-item label="作业ID" prop="homeworkId">
            <el-input 
              placeholder="请输入作业ID" 
              v-model="state.queryForm.homeworkId" 
            />
          </el-form-item>
          <el-form-item label="提交学生ID" prop="studentId">
            <el-input 
              placeholder="请输入提交学生ID" 
              v-model="state.queryForm.studentId" 
            />
          </el-form-item>
          <el-form-item label="状态" prop="status">
            <el-select v-model="state.queryForm.status" placeholder="请选择作业状态" clearable>
              <el-option label="未交" value="0" />
              <el-option label="已交待批" value="1" />
              <el-option label="已批阅" value="2" />
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button icon="search" type="primary" @click="getDataList">
              查询
            </el-button>
            <el-button icon="Refresh" @click="resetQuery">重置</el-button>
          </el-form-item>
        </el-form>
      </el-row>

      <!-- 操作按钮区域 -->
      <el-row>
        <div class="mb8" style="width: 100%">
          <el-button 
            icon="folder-add" 
            type="primary" 
            class="ml10" 
            @click="formDialogRef.openDialog()"
            v-auth="'homework_homeworkSubmission_add'"
          >
            新增
          </el-button>
          <el-button 
            plain 
            icon="upload-filled" 
            type="primary" 
            class="ml10" 
            @click="excelUploadRef.show()" 
            v-auth="'homework_homeworkSubmission_add'"
          >
            导入
          </el-button>
          <el-button 
            plain 
            :disabled="multiple" 
            icon="Delete" 
            type="primary"
            v-auth="'homework_homeworkSubmission_del'" 
            @click="handleDelete(selectObjs.value)"
          >
            删除
          </el-button>
          <right-toolbar 
            v-model:showSearch="showSearch" 
            :export="'homework_homeworkSubmission_export'"
            @exportExcel="exportExcel" 
            class="ml10 mr20" 
            style="float: right;"
            @queryTable="getDataList"
          />
        </div>
      </el-row>

      <!-- 数据表格区域 -->
      <el-table 
        :data="state.dataList" 
        v-loading="state.loading" 
        border 
        :cell-style="tableStyle.cellStyle" 
        :header-cell-style="tableStyle.headerCellStyle"
        @selection-change="selectionChangHandle"
        @sort-change="sortChangeHandle"
      >
        <el-table-column type="selection" width="40" align="center" />
        <el-table-column type="index" label="#" width="40" />
        <el-table-column 
          prop="studentId" 
          label="提交学生ID" 
          show-overflow-tooltip
        />
        <el-table-column 
          prop="fileUrl" 
          label="作业文件URL" 
          show-overflow-tooltip
        />
        <el-table-column 
          prop="score" 
          label="教师打分" 
          show-overflow-tooltip
        />
        <el-table-column prop="status" label="状态" show-overflow-tooltip>
          <template #default="scope">
            <el-tag v-if="scope.row.status === '0'" type="info">未交</el-tag>
            <el-tag v-else-if="scope.row.status === '1'" type="warning">已交待批</el-tag>
            <el-tag v-else-if="scope.row.status === '2'" type="success">已批阅</el-tag>
          </template>
        </el-table-column>
        <el-table-column 
          prop="submitTime" 
          label="实际提交时间" 
          sortable="custom" 
          show-overflow-tooltip
        />
        <el-table-column label="操作" width="150">
          <template #default="scope">
            <el-button 
              icon="edit-pen" 
              text 
              type="primary" 
              v-auth="'homework_homeworkSubmission_edit'"
              @click="formDialogRef.openDialog(scope.row.submissionId)"
            >
              编辑
            </el-button>
            <el-button 
              icon="delete" 
              text 
              type="primary" 
              v-auth="'homework_homeworkSubmission_del'" 
              @click="handleDelete([scope.row.submissionId])"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页组件 -->
      <pagination 
        @size-change="sizeChangeHandle" 
        @current-change="currentChangeHandle" 
        v-bind="state.pagination" 
      />
    </div>

    <!-- 编辑、新增弹窗 -->
    <form-dialog ref="formDialogRef" @refresh="getDataList(false)" />

    <!-- 导入excel弹窗 (需要在 upms-biz/resources/file 下维护模板) -->
    <upload-excel
      ref="excelUploadRef"
      title="导入"
      url="/homework/homeworkSubmission/import"
      temp-url="/admin/sys-file/local/file/homeworkSubmission.xlsx"
      @refreshDataList="getDataList"
    />
  </div>
</template>

<script setup lang="ts" name="systemHomeworkSubmission">
// ========== 导入声明 ==========
import { BasicTableProps, useTable } from "/@/hooks/table";
import { fetchList, delObjs } from "/@/api/homework/homeworkSubmission";
import { useMessage, useMessageBox } from "/@/hooks/message";
import { useDict } from '/@/hooks/dict';

// ========== 组件声明 ==========
// 异步加载表单弹窗组件
const FormDialog = defineAsyncComponent(() => import('./form.vue'));

// ========== 字典数据 ==========

// ========== 组件引用 ==========
const formDialogRef = ref();          // 表单弹窗引用
const excelUploadRef = ref();         // Excel上传弹窗引用
const queryRef = ref();               // 查询表单引用

// ========== 响应式数据 ==========
const showSearch = ref(true);         // 是否显示搜索区域
const selectObjs = ref([]) as any;    // 表格多选数据
const multiple = ref(true);           // 是否多选

// ========== 表格状态 ==========
const state: BasicTableProps = reactive<BasicTableProps>({
  queryForm: {},    // 查询参数
  pageList: fetchList // 分页查询方法
});

// ========== Hook引用 ==========
// 表格相关Hook
const {
  getDataList,
  currentChangeHandle,
  sizeChangeHandle,
  sortChangeHandle,
  downBlobFile,
  tableStyle
} = useTable(state);

// ========== 方法定义 ==========
/**
 * 重置查询条件
 */
const resetQuery = () => {
  // 清空搜索条件
  queryRef.value?.resetFields();
  // 清空多选
  selectObjs.value = [];
  // 重新查询
  getDataList();
};

/**
 * 导出Excel文件
 */
const exportExcel = () => {
  downBlobFile(
    '/homework/homeworkSubmission/export',
    { ...state.queryForm, ids: selectObjs.value },
    'homeworkSubmission.xlsx'
  );
};

/**
 * 表格多选事件处理
 * @param objs 选中的数据行
 */
const selectionChangHandle = (objs: { submissionId: string }[]) => {
  selectObjs.value = objs.map(({ submissionId }) => submissionId);
  multiple.value = !objs.length;
};

/**
 * 删除数据处理
 * @param ids 要删除的数据ID数组
 */
const handleDelete = async (ids: string[]) => {
  try {
    await useMessageBox().confirm('此操作将永久删除');
  } catch {
    return;
  }

  try {
    await delObjs(ids);
    getDataList();
    useMessage().success('删除成功');
  } catch (err: any) {
    useMessage().error(err.msg);
  }
};
</script>
