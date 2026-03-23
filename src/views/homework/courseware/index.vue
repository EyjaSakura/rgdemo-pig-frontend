<template>
  <div class="layout-padding">
    <div class="layout-padding-auto layout-padding-view">
      <!-- 查询表单区域 -->
      <el-row v-show="showSearch">
        <el-form :model="state.queryForm" ref="queryRef" :inline="true" @keyup.enter="getDataList">
          <el-form-item label="所属课程ID" prop="courseId">
            <el-input 
              placeholder="请输入所属课程ID" 
              v-model="state.queryForm.courseId" 
            />
          </el-form-item>
          <el-form-item label="文件夹/分组名称" prop="folderName">
            <el-input 
              placeholder="请输入文件夹/分组名称" 
              v-model="state.queryForm.folderName" 
            />
          </el-form-item>
          <el-form-item label="课件命名" prop="title">
            <el-input 
              placeholder="请输入课件命名" 
              v-model="state.queryForm.title" 
            />
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
            v-auth="'homework_courseware_add'"
          >
            新增
          </el-button>
          <el-button 
            plain 
            icon="upload-filled" 
            type="primary" 
            class="ml10" 
            @click="excelUploadRef.show()" 
            v-auth="'homework_courseware_add'"
          >
            导入
          </el-button>
          <el-button 
            plain 
            :disabled="multiple" 
            icon="Delete" 
            type="primary"
            v-auth="'homework_courseware_del'" 
            @click="handleDelete(selectObjs.value)"
          >
            删除
          </el-button>
          <right-toolbar 
            v-model:showSearch="showSearch" 
            :export="'homework_courseware_export'"
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
          prop="folderName" 
          label="文件夹/分组名称" 
          show-overflow-tooltip
        />
        <el-table-column 
          prop="title" 
          label="课件命名" 
          show-overflow-tooltip
        />
        <el-table-column 
          prop="fileUrl" 
          label="课件文件下载地址/OSS路径" 
          show-overflow-tooltip
        />
        <el-table-column 
          prop="sortOrder" 
          label="排序权重(用于调整顺序)" 
          show-overflow-tooltip
        />
        <el-table-column 
          prop="uploadTime" 
          label="上传时间" 
          sortable="custom" 
          show-overflow-tooltip
        />
        <el-table-column label="操作" width="150">
          <template #default="scope">
            <el-button 
              icon="edit-pen" 
              text 
              type="primary" 
              v-auth="'homework_courseware_edit'"
              @click="formDialogRef.openDialog(scope.row.coursewareId)"
            >
              编辑
            </el-button>
            <el-button 
              icon="delete" 
              text 
              type="primary" 
              v-auth="'homework_courseware_del'" 
              @click="handleDelete([scope.row.coursewareId])"
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
      url="/homework/courseware/import"
      temp-url="/admin/sys-file/local/file/courseware.xlsx"
      @refreshDataList="getDataList"
    />
  </div>
</template>

<script setup lang="ts" name="systemCourseware">
// ========== 导入声明 ==========
import { BasicTableProps, useTable } from "/@/hooks/table";
import { fetchList, delObjs } from "/@/api/homework/courseware";
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
    '/homework/courseware/export',
    { ...state.queryForm, ids: selectObjs.value },
    'courseware.xlsx'
  );
};

/**
 * 表格多选事件处理
 * @param objs 选中的数据行
 */
const selectionChangHandle = (objs: { coursewareId: string }[]) => {
  selectObjs.value = objs.map(({ coursewareId }) => coursewareId);
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
