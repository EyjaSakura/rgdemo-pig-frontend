<template>
  <el-dialog :title="form.homeworkId ? '编辑' : '新增'" v-model="visible"
    :close-on-click-modal="false" draggable>
    <el-form ref="dataFormRef" :model="form" :rules="dataRules" formDialogRef label-width="90px" v-loading="loading">
      <el-row :gutter="24">
        <el-col :span="12" class="mb20">
          <el-form-item label="所属课程ID" prop="courseId">
            <el-input v-model="form.courseId" placeholder="请输入所属课程ID"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="作业名" prop="title">
            <el-input v-model="form.title" placeholder="请输入作业名"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="作业描述" prop="description">
            <el-input type="textarea" v-model="form.description" placeholder="请输入作业描述"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="教师上传的附件地址" prop="attachmentUrl">
            <upload-file v-model="form.attachmentUrl"></upload-file>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="发布时间" prop="publishTime">
            <el-date-picker type="datetime" placeholder="请选择发布时间" v-model="form.publishTime" :value-format="dateTimeStr"></el-date-picker>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="提交截止时间" prop="deadline">
            <el-date-picker type="datetime" placeholder="请选择提交截止时间" v-model="form.deadline" :value-format="dateTimeStr"></el-date-picker>
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

<script setup lang="ts" name="HomeworkDialog">
// ========== 1. 导入语句 ==========
import { useDict } from '/@/hooks/dict';
import { rule } from '/@/utils/validate';
import { useMessage } from "/@/hooks/message";
import { getObj, addObj, putObj, validateExist } from '/@/api/homework/homework';

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
  homeworkId: '', // 主键
  courseId: '', // 所属课程ID
  title: '', // 作业名
  description: '', // 作业描述
  attachmentUrl: '', // 教师上传的附件地址
  publishTime: '', // 发布时间
  deadline: '', // 提交截止时间
});

// ========== 4. 字典数据处理 ==========

// ========== 5. 表单校验规则 ==========
const dataRules = ref({
  courseId: [
    { required: true, message: '所属课程ID不能为空', trigger: 'blur' }
  ],
  title: [
    { required: true, message: '作业名不能为空', trigger: 'blur' }
  ],
  description: [
    { required: true, message: '作业描述不能为空', trigger: 'blur' }
  ],
  deadline: [
    { required: true, message: '提交截止时间不能为空', trigger: 'blur' }
  ],
});

// ========== 6. 方法定义 ==========
// 获取详情数据
const getHomeworkData = async (id: string) => {
  try {
    loading.value = true;
    const { data } = await getObj({ homeworkId: id });
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
  form.homeworkId = '';

  // 重置表单数据
  nextTick(() => {
    dataFormRef.value?.resetFields();
  });

  // 获取Homework信息
  if (id) {
    form.homeworkId = id;
    getHomeworkData(id);
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
    form.homeworkId ? await putObj(form) : await addObj(form);
    useMessage().success(form.homeworkId ? '修改成功' : '添加成功');
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