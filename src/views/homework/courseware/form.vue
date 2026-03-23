<template>
  <el-dialog :title="form.coursewareId ? '编辑' : '新增'" v-model="visible"
    :close-on-click-modal="false" draggable>
    <el-form ref="dataFormRef" :model="form" :rules="dataRules" formDialogRef label-width="90px" v-loading="loading">
      <el-row :gutter="24">
        <el-col :span="12" class="mb20">
          <el-form-item label="所属课程ID" prop="courseId">
            <el-input v-model="form.courseId" placeholder="请输入所属课程ID"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="文件夹/分组名称" prop="folderName">
            <el-input v-model="form.folderName" placeholder="请输入文件夹/分组名称"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="课件命名" prop="title">
            <el-input v-model="form.title" placeholder="请输入课件命名"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="课件文件下载地址/OSS路径" prop="fileUrl">
            <upload-file v-model="form.fileUrl"></upload-file>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="排序权重(用于调整顺序)" prop="sortOrder">
            <el-input-number :min="1" :max="1000" v-model="form.sortOrder" placeholder="请输入排序权重(用于调整顺序)"></el-input-number>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="onSubmit" :disabled="loading">确 认</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup lang="ts" name="CoursewareDialog">
// ========== 1. 导入语句 ==========
import { useDict } from '/@/hooks/dict';
import { rule } from '/@/utils/validate';
import { useMessage } from "/@/hooks/message";
import { getObj, addObj, putObj, validateExist } from '/@/api/homework/courseware';

// ========== 2. 组件定义 ==========
// 定义组件事件
const emit = defineEmits(['refresh']);

// ========== 3. 响应式数据定义 ==========
// 基础响应式变量
const dataFormRef = ref(); // 表单引用
const visible = ref(false); // 弹窗显示状态
const loading = ref(false); // 加载状态

// 表单数据对象
const form = reactive({
  coursewareId: '', // 主键
  courseId: '', // 所属课程ID
  folderName: '', // 文件夹/分组名称
  title: '', // 课件命名
  fileUrl: '', // 课件文件下载地址/OSS路径
  sortOrder: 0, // 排序权重(用于调整顺序)
});

// ========== 4. 字典数据处理 ==========

// ========== 5. 表单校验规则 ==========
const dataRules = ref({
  courseId: [
    { required: true, message: '所属课程ID不能为空', trigger: 'blur' }
  ],
  folderName: [
    { required: true, message: '文件夹/分组名称不能为空', trigger: 'blur' }
  ],
  title: [
    { required: true, message: '课件命名不能为空', trigger: 'blur' }
  ],
  fileUrl: [
    { required: true, message: '课件文件下载地址/OSS路径不能为空', trigger: 'blur' }
  ],
});

// ========== 6. 方法定义 ==========
// 获取详情数据
const getCoursewareData = async (id: string) => {
  try {
    loading.value = true;
    const { data } = await getObj({ coursewareId: id });
    // 直接将第一条数据赋值给表单
    Object.assign(form, data[0]);
  } catch (error) {
    useMessage().error('获取数据失败');
  } finally {
    loading.value = false;
  }
};

// 打开弹窗方法
const openDialog = (id: string) => {
  visible.value = true;
  form.coursewareId = '';

  // 重置表单数据
  nextTick(() => {
    dataFormRef.value?.resetFields();
  });

  // 获取Courseware信息
  if (id) {
    form.coursewareId = id;
    getCoursewareData(id);
  }
};

// 提交表单方法
const onSubmit = async () => {
  loading.value = true; // 防止重复提交
  
  // 表单校验
  const valid = await dataFormRef.value.validate().catch(() => {});
  if (!valid) {
    loading.value = false;
    return false;
  }

  try {
    // 根据是否有ID判断是新增还是修改
    form.coursewareId ? await putObj(form) : await addObj(form);
    useMessage().success(form.coursewareId ? '修改成功' : '添加成功');
    visible.value = false;
    emit('refresh'); // 通知父组件刷新列表
  } catch (err: any) {
    useMessage().error(err.msg);
  } finally {
    loading.value = false;
  }
};

// ========== 7. 对外暴露 ==========
// 暴露方法给父组件
defineExpose({
  openDialog
});
</script> 