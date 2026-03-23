<template>
  <el-dialog :title="isStudent ? '提交作业' : '批改作业'" v-model="visible" :close-on-click-modal="false" draggable>
    <el-form ref="dataFormRef" :model="form" :rules="dataRules" label-width="100px" v-loading="loading">
      <el-row :gutter="24">
        <el-col :span="12" class="mb20">
          <el-form-item label="关联作业" prop="homeworkId">
            <el-input v-model="form.homeworkId" placeholder="关联的作业ID" :disabled="!isStudent" />
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="提交学生" prop="studentId">
            <el-input v-model="form.studentId" placeholder="学生ID" disabled />
          </el-form-item>
        </el-col>

        <el-col :span="24" class="mb20">
          <el-form-item label="作业文件" prop="fileUrl">
            <upload-file v-if="isStudent" v-model="form.fileUrl"></upload-file>
            <div v-else>
              <el-link v-if="form.fileUrl" :href="form.fileUrl" type="primary" target="_blank" icon="view">
                点击预览/下载学生作业
              </el-link>
              <el-tag v-else type="info">未上传文件</el-tag>
            </div>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20" v-if="isTeacher">
          <el-form-item label="作业评分" prop="score">
            <el-input-number :min="0" :max="100" v-model="form.score" placeholder="0-100" />
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="作业状态" prop="status">
            <el-select v-model="form.status" placeholder="请选择状态" :disabled="isStudent">
              <el-option label="未交" value="0" />
              <el-option label="已交待批" value="1" />
              <el-option label="已批阅" value="2" />
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="onSubmit" :disabled="loading">
          {{ isStudent ? '提交作业' : '提交批改' }}
        </el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup lang="ts" name="HomeworkSubmissionDialog">
import { useMessage } from "/@/hooks/message";
import { getObj, addObj, putObj } from '/@/api/homework/homeworkSubmission';
import { auth } from '/@/utils/authFunction'; // 导入权限判断工具

const emit = defineEmits(['refresh']);

// ========== 身份逻辑判断 ==========
// 拥有 add 权限的通常是学生角色，拥有 edit 权限的通常是教师角色
const isStudent = computed(() => auth('homework_homeworkSubmission_add'));
const isTeacher = computed(() => auth('homework_homeworkSubmission_edit'));

const dataFormRef = ref();
const visible = ref(false);
const loading = ref(false);

const form = reactive({
  submissionId: '',
  homeworkId: '',
  studentId: '',
  fileUrl: '',
  score: 0,
  status: '0', // 默认未交
});

const dataRules = ref({
  homeworkId: [{ required: true, message: '请选择关联的作业', trigger: 'blur' }],
  fileUrl: [{ required: true, message: '请上传作业文件', trigger: 'change' }],
});

// 获取详情
const getHomeworkSubmissionData = async (id: string) => {
  try {
    loading.value = true;
    const { data } = await getObj({ submissionId: id });
    Object.assign(form, data[0]);
  } catch (error) {
    useMessage().error('获取数据失败');
  } finally {
    loading.value = false;
  }
};

// 打开弹窗
const openDialog = (id: string) => {
  visible.value = true;
  // 重置表单
  resetForm();

  if (id) {
    form.submissionId = id;
    getHomeworkSubmissionData(id);
  } else {
    // 如果是学生新增，默认状态设为“已交待批”
    if(isStudent.value) {
      form.status = '1';
    }
  }
};

const resetForm = () => {
  form.submissionId = '';
  form.homeworkId = '';
  form.studentId = ''; // 实际项目中这里通常从 store 获取当前登录用户ID填充
  form.fileUrl = '';
  form.score = 0;
  form.status = '0';
  nextTick(() => {
    dataFormRef.value?.resetFields();
  });
};

// 提交
const onSubmit = async () => {
  const valid = await dataFormRef.value.validate().catch(() => {});
  if (!valid) return;

  try {
    loading.value = true;
    // 业务逻辑微调：
    // 如果是学生提交，状态强制为 1
    // 如果是教师修改且打了分，状态通常改为 2
    if (isTeacher.value && form.score > 0) {
        form.status = '2';
    }

    form.submissionId ? await putObj(form) : await addObj(form);
    useMessage().success('操作成功');
    visible.value = false;
    emit('refresh');
  } catch (err: any) {
    useMessage().error(err.msg || '操作失败');
  } finally {
    loading.value = false;
  }
};

defineExpose({ openDialog });
</script>