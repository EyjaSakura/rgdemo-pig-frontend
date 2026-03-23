<template>
  <el-dialog :title="form.id ? '编辑' : '新增'" v-model="visible"
    :close-on-click-modal="false" draggable>
    <el-form ref="dataFormRef" :model="form" :rules="dataRules" formDialogRef label-width="90px" v-loading="loading">
      <el-row :gutter="24">
        <el-col :span="12" class="mb20">
          <el-form-item label="班级ID (关联 sys_dept.dept_id)" prop="deptId">
            <el-select v-model="form.deptId" placeholder="请选择班级ID (关联 sys_dept.dept_id)">
              <el-option label="请选择" value="0"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="学生ID (关联 sys_user.user_id)" prop="studentId">
            <el-select v-model="form.studentId" placeholder="请选择学生ID (关联 sys_user.user_id)">
              <el-option label="请选择" value="0"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12" class="mb20">
          <el-form-item label="职务名称(如:班长,心理委员等)" prop="roleName">
            <el-input v-model="form.roleName" placeholder="请输入职务名称(如:班长,心理委员等)"/>
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

<script setup lang="ts" name="ClassCommitteeDialog">
// ========== 1. 导入语句 ==========
import { useDict } from '/@/hooks/dict';
import { rule } from '/@/utils/validate';
import { useMessage } from "/@/hooks/message";
import { getObj, addObj, putObj, validateExist } from '/@/api/homework/classCommittee';

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
  id: '', // 主键
  deptId: '', // 班级ID (关联 sys_dept.dept_id)
  studentId: '', // 学生ID (关联 sys_user.user_id)
  roleName: '', // 职务名称(如:班长,心理委员等)
});

// ========== 4. 字典数据处理 ==========

// ========== 5. 表单校验规则 ==========
const dataRules = ref({
  deptId: [
    { required: true, message: '班级ID (关联 sys_dept.dept_id)不能为空', trigger: 'blur' }
  ],
  studentId: [
    { required: true, message: '学生ID (关联 sys_user.user_id)不能为空', trigger: 'blur' }
  ],
  roleName: [
    { required: true, message: '职务名称(如:班长,心理委员等)不能为空', trigger: 'blur' }
  ],
});

// ========== 6. 方法定义 ==========
// 获取详情数据
const getClassCommitteeData = async (id: string) => {
  try {
    loading.value = true;
    const { data } = await getObj({ id: id });
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
  form.id = '';

  // 重置表单数据
  nextTick(() => {
    dataFormRef.value?.resetFields();
  });

  // 获取ClassCommittee信息
  if (id) {
    form.id = id;
    getClassCommitteeData(id);
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
    form.id ? await putObj(form) : await addObj(form);
    useMessage().success(form.id ? '修改成功' : '添加成功');
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