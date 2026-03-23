<template>
  <el-dialog :title="form.courseId ? '编辑' : '新增'" v-model="visible"
    :close-on-click-modal="false" draggable>
    <el-form ref="dataFormRef" :model="form" :rules="dataRules" formDialogRef label-width="90px" v-loading="loading">
      <el-row :gutter="24">
        <el-col :span="12" class="mb20">
          <el-form-item label="课程名" prop="courseName">
            <el-input v-model="form.courseName" placeholder="请输入课程名"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="课程号" prop="courseCode">
            <el-input v-model="form.courseCode" placeholder="请输入课程号"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="课程学分" prop="credit">
            <el-input-number :min="1" :max="1000" v-model="form.credit" placeholder="请输入课程学分"></el-input-number>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="上课时间与地点" prop="timePlace">
            <el-input v-model="form.timePlace" placeholder="请输入上课时间与地点"/>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="是否必修 (0选修 1必修)" prop="isRequired">
            <el-radio-group v-model="form.isRequired">
              <el-radio label="是否必修 (0选修 1必修)" border>是否必修 (0选修 1必修)</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="状态 (0正常 1已结课)" prop="status">
            <el-radio-group v-model="form.status">
              <el-radio label="状态 (0正常 1已结课)" border>状态 (0正常 1已结课)</el-radio>
            </el-radio-group>
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

<script setup lang="ts" name="CourseDialog">
// ========== 1. 导入语句 ==========
import { useDict } from '/@/hooks/dict';
import { rule } from '/@/utils/validate';
import { useMessage } from "/@/hooks/message";
import { getObj, addObj, putObj, validateExist } from '/@/api/homework/course';

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
  courseId: '', // 主键
  courseName: '', // 课程名
  courseCode: '', // 课程号
  credit: 0, // 课程学分
  timePlace: '', // 上课时间与地点
  isRequired: '', // 是否必修 (0选修 1必修)
  status: '', // 状态 (0正常 1已结课)
});

// ========== 4. 字典数据处理 ==========

// ========== 5. 表单校验规则 ==========
const dataRules = ref({
  courseName: [
    { required: true, message: '课程名不能为空', trigger: 'blur' }
  ],
  courseCode: [
    { required: true, message: '课程号不能为空', trigger: 'blur' }
  ],
  credit: [
    { required: true, message: '课程学分不能为空', trigger: 'blur' }
  ],
  isRequired: [
    { required: true, message: '是否必修 (0选修 1必修)不能为空', trigger: 'blur' }
  ],
});

// ========== 6. 方法定义 ==========
// 获取详情数据
const getCourseData = async (id: string) => {
  try {
    loading.value = true;
    const { data } = await getObj({ courseId: id });
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
  form.courseId = '';

  // 重置表单数据
  nextTick(() => {
    dataFormRef.value?.resetFields();
  });

  // 获取Course信息
  if (id) {
    form.courseId = id;
    getCourseData(id);
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
    form.courseId ? await putObj(form) : await addObj(form);
    useMessage().success(form.courseId ? '修改成功' : '添加成功');
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