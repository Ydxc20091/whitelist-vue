<template>
  <div
      class="app-wrapper"
      @click.self="handleFormBlur"
      @touchstart.self="handleFormBlur"
  >
    <SakuraBackground
        :currentTheme="currentTheme"
    />
    <div
        class="form-container"
        :class="{ 'focused': isFormFocused }"
        @click="handleFormFocus"
        @touchstart="handleFormFocus"
    >
      <div class="title-container">
        <i class="el-icon-user-solid"></i>
        <h2>NodeMC入服白名单申请</h2>
        <el-button
            class="view-members-btn"
            text
            type="primary"
            @click="$router.push('/whitelist-members')"
        >
          <el-icon>
            <User/>
          </el-icon>
          查看成员
        </el-button>
      </div>

      <div class="description">
        欢迎加入我们的服务器！请填写以下信息完成白名单申请。
      </div>

      <el-form :model="form" class="animated-form" label-width="auto">
        <el-form-item label="ID：">
          <el-input v-model="form.userName" placeholder="请输入游戏名称"></el-input>
        </el-form-item>
        <el-form-item label="QQ：">
          <el-input v-model="form.qqNum" placeholder="请输入QQ号"></el-input>
        </el-form-item>
        <el-form-item label="正版：">
          <el-radio-group v-model="form.onlineFlag">
            <el-radio label="1">是</el-radio>
            <el-radio label="0">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="描述：" label-width="auto">
          <el-input v-model="form.remark" placeholder="请输入描述 非必填" type="textarea"></el-input>
        </el-form-item>
        <div class="button-group">
          <el-button v-loading.fullscreen.lock="fullscreenLoading"
                     class="submit-btn"
                     type="primary"
                     @click="submitForm">
            <i class="el-icon-check"></i> 提交申请
          </el-button>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue';
import { ElMessage } from 'element-plus';
import axios from 'axios';
import { User } from '@element-plus/icons-vue';
import SakuraBackground from './common/SakuraBackground.vue';
import { addIPToHeaders } from '../utils/ipUtils';

const http = axios.create({
  baseURL: import.meta.env.VITE_API_URL, // 使用环境变量
  timeout: 8000
});

const form = reactive({
  userName: '',
  qqNum: '',
  onlineFlag: '',
  remark: ''
});

// 添加全屏loading状态
const fullscreenLoading = ref(false);

const submitForm = async () => {
  if (!form.userName || !form.qqNum || !form.onlineFlag) {
    ElMessage.error('请填写完整信息');
  } else if (!/^\d{5,11}$/.test(form.qqNum)) {
    ElMessage.error('QQ号格式错误');
  } else {
    fullscreenLoading.value = true;

    try {
      // 使用封装的IP工具函数获取请求头
      const headers = await addIPToHeaders();
      
      // 发送表单请求
      const res = await http.post('/mc/whitelist/apply', form, {headers});
      
      if (res.data.code === 200) {
        ElMessage.success(res.data.msg);
      } else {
        ElMessage.error(res.data.msg || '未知错误，请联系管理员');
      }
    } catch (error) {
      console.error('提交表单请求出错：', error);
      ElMessage.error('提交表单时发生错误，请检查网络或联系管理员');
    } finally {
      fullscreenLoading.value = false;
    }
  }
};

// 获取当前主题
const currentTheme = ref(localStorage.getItem('theme') || 'default')

const isFormFocused = ref(false);

const handleFormFocus = () => {
  isFormFocused.value = true;
};

const handleFormBlur = () => {
  isFormFocused.value = false;
};

</script>

<style scoped>
.app-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  min-height: 100vh;
  background-size: 400% 400%;
  background-image: var(--theme-gradient, linear-gradient(-45deg, #e0e0ff, #ffd6e7, #ffdfd0, #fff4d1, #d4ffe6, #cce9ff, #f0e6ff, #ffe6f0, #ffe6e6));
  animation: warmGradient 20s ease infinite;
  font-family: 'CustomFont', sans-serif;
  transition: background-image 0.5s ease;
}

@keyframes warmGradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.form-container {
  background: var(--theme-bg);
  color: var(--theme-text);
  backdrop-filter: blur(8px);
  padding: 30px;
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  width: min(100%, 500px);
  transform: translateY(0);
  transition: all 0.3s ease;
}

.form-container:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.title-container {
  text-align: center;
  margin-bottom: 25px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 10px;
}

.title-container h2 {
  color: var(--theme-primary);
  font-size: 28px;
  margin: 10px 0;
  flex-grow: 0; /* 防止标题占据过多空间 */
}

.description {
  text-align: center;
  color: #333;
  margin-bottom: 25px;
  font-size: 14px;
  font-weight: 500;
}

.animated-form :deep(.el-form-item) {
  transition: all 0.3s ease;
  margin-bottom: 25px;
}

.animated-form :deep(.el-form-item:hover) {
  transform: translateX(5px);
}

.button-group {
  display: flex;
  justify-content: center;
  margin-top: 30px;
}

.submit-btn {
  min-width: 140px;
  height: 40px;
  border-radius: 20px;
  font-weight: 500;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
  font-family: 'CustomFont', sans-serif;
  background: linear-gradient(45deg, #409EFF, #36cfc9);
  border: none;
  box-shadow: 0 4px 15px rgba(64, 158, 255, 0.3);
  z-index: 1;
}

.submit-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
      90deg,
      transparent,
      rgba(255, 255, 255, 0.2),
      transparent
  );
  transition: 0.5s;
  z-index: -1;
}

.submit-btn:hover::before {
  left: 100%;
}

.submit-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(64, 158, 255, 0.4);
}

.submit-btn:active {
  transform: scale(0.98);
}

:deep(.el-form-item__label) {
  font-weight: 500;
  color: #303133;
  transition: all 0.3s ease;
}

:deep(.el-form-item:hover .el-form-item__label) {
  color: var(--theme-primary);
  transform: translateX(2px);
}

:deep(.el-input__inner) {
  border-radius: 8px;
  border: 1px solid rgba(64, 158, 255, 0.2);
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.8);
  height: 40px;
  transform: translateY(0);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-family: 'CustomFont', sans-serif;
}

:deep(.el-input__inner:hover) {
  border-color: #409EFF;
  box-shadow: 0 2px 8px rgba(64, 158, 255, 0.1);
}

:deep(.el-input__inner:focus) {
  border-color: #409EFF;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.1);
  background: #fff;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(64, 158, 255, 0.15);
}

:deep(.el-textarea__inner) {
  border-radius: 8px;
  border: 1px solid rgba(64, 158, 255, 0.2);
  transition: all 0.3s ease;
  background: rgba(255, 255, 255, 0.8);
  min-height: 100px;
  padding: 12px;
}

:deep(.el-textarea__inner:hover) {
  border-color: #409EFF;
}

:deep(.el-textarea__inner:focus) {
  border-color: #409EFF;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.1);
  background: #fff;
}

.form-container :deep(.el-input__wrapper),
.form-container :deep(.el-textarea__wrapper) {
  box-shadow: none !important;
  padding: 0;
  background: none;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.form-container {
  animation: fadeIn 0.6s ease-out;
}

/* 优化表单容器 */
.form-container {
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

/* 优化提交按钮 */
.submit-btn {
  background: linear-gradient(45deg, #409EFF, #36cfc9);
  border: none;
  box-shadow: 0 4px 15px rgba(64, 158, 255, 0.3);
}

.submit-btn:hover {
  background: linear-gradient(45deg, #66b1ff, #5cdbd3);
  box-shadow: 0 6px 20px rgba(64, 158, 255, 0.4);
}

/* 优化单选按钮组样式 */
:deep(.el-radio) {
  margin-right: 20px;
  transition: all 0.3s ease;
}

:deep(.el-radio:hover) {
  transform: translateY(-2px);
}

:deep(.el-radio__input.is-checked + .el-radio__label) {
  color: #409EFF;
  font-weight: 500;
}

/* 樱花容器 */
.sakura-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

/* 樱花样式 */
.sakura {
  position: absolute;
  top: -10%;
  left: var(--left);
  width: var(--size);
  height: var(--size);
  background: #ffd7e6;
  border-radius: 100% 0 100% 100%;
  animation: fall var(--delay) linear infinite;
  transform-origin: center;
  opacity: 0.7;
}

.sakura::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: inherit;
  border-radius: inherit;
  transform: rotate(45deg);
}

/* 樱花飘落动画 */
@keyframes fall {
  0% {
    top: -10%;
    transform: rotate(0deg) translateX(0);
    opacity: 0;
  }
  10% {
    opacity: 0.7;
  }
  90% {
    opacity: 0.7;
  }
  100% {
    top: 100%;
    transform: rotate(360deg) translateX(100px);
    opacity: 0;
  }
}

/* 修改表单容器的z-index确保在樱花上层 */
.form-container {
  z-index: 1001;
  position: relative;
}

/* 修改查看成员按钮样式 */
.view-members-btn {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  color: var(--theme-primary);
  transition: all 0.3s ease;
  padding: 6px 12px;
  border-radius: 15px;
  background: rgba(var(--theme-primary-rgb), 0.1);
  font-family: 'CustomFont', sans-serif;
}

.view-members-btn:hover {
  color: var(--theme-secondary);
  transform: translateY(-50%) translateX(-2px);
  background: rgba(var(--theme-primary-rgb), 0.15);
}

.view-members-btn .el-icon {
  font-size: 16px;
}

/* 主题特定样式 */
[data-theme="sakura"] .title-container h2 {
  text-shadow: 0 0 10px rgba(255, 105, 180, 0.3);
}

[data-theme="sakura"] .view-members-btn:hover {
  text-shadow: 0 0 8px rgba(255, 105, 180, 0.3);
}

/* 暗色模式下的标题和按钮颜色 */
.dark .title-container h2,
.dark .view-members-btn {
  color: var(--theme-text-dark);
}

.dark .view-members-btn:hover {
  color: var(--theme-secondary);
}

/* 暗色模式样式 */
html.dark .form-container {
  background-color: rgba(30, 30, 40, 0.75);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

html.dark .form-container:hover {
  box-shadow: 0 12px 40px rgba(64, 158, 255, 0.2);
  border: 1px solid rgba(64, 158, 255, 0.2);
}

html.dark .title-container h2 {
  text-shadow: 0 0 10px rgba(64, 158, 255, 0.3);
}

html.dark .description {
  color: rgba(255, 255, 255, 0.8);
}

html.dark .submit-btn {
  background: linear-gradient(45deg, #0a84ff, #00b3e6);
  box-shadow: 0 4px 15px rgba(10, 132, 255, 0.3);
}

html.dark .submit-btn:hover {
  box-shadow: 0 6px 20px rgba(10, 132, 255, 0.5);
}

/* 为表单项添加微交互 */
.el-form-item {
  position: relative;
}

.animated-form :deep(.el-input),
.animated-form :deep(.el-textarea) {
  position: relative;
  z-index: 1;
}

.animated-form :deep(.el-input::after),
.animated-form :deep(.el-textarea::after) {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 50%;
  width: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, var(--theme-primary), transparent);
  transition: width 0.3s ease, left 0.3s ease;
  z-index: 0;
}

.animated-form :deep(.el-input:focus-within::after),
.animated-form :deep(.el-textarea:focus-within::after) {
  width: 100%;
  left: 0%;
}

/* 修改移动端样式 */
@media (max-width: 768px) {
  .app-wrapper {
    flex-direction: column;
    padding: 20px;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }

  .form-container {
    margin: 0;
    width: 100%;
    max-width: 500px;
    animation: fadeIn 0.5s ease-out;
    box-shadow: 0 15px 30px rgba(0, 0, 0, 0.15);
    backdrop-filter: blur(12px);
    border-radius: 20px;
    transition: transform 0.3s ease, box-shadow 0.3s ease;
  }

  /* 移动端表单容器适配 */
  .form-container:hover {
    transform: none;
  }

  /* 添加表单聚焦效果 */
  .form-container.focused {
    transform: translateY(-10px);
    box-shadow: 0 20px 40px rgba(var(--theme-primary-rgb), 0.2);
  }

  /* 修改查看成员按钮样式在移动端的表现 */
  .view-members-btn {
    font-size: 12px;
    padding: 4px 8px;
  }

  /* 优化表单元素在移动端的显示 */
  :deep(.el-form-item__label) {
    padding-right: 8px;
  }

  :deep(.el-radio) {
    margin-right: 12px;
  }

  .animated-form :deep(.el-form-item:hover) {
    transform: none;
  }

  /* 美化提交按钮 */
  .submit-btn {
    transform: scale(1);
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    width: 80%;
    margin: 0 auto;
    border-radius: 12px;
    background: linear-gradient(45deg, #409EFF, #36cfc9);
    box-shadow: 0 4px 15px rgba(64, 158, 255, 0.2);
  }

  .submit-btn:hover, .submit-btn:active {
    transform: translateY(-3px);
    box-shadow: 0 8px 20px rgba(64, 158, 255, 0.4);
  }
}

/* 添加表单标题动画效果 */
.title-container h2 {
  position: relative;
  overflow: hidden;
}

.title-container h2::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, var(--theme-primary), transparent);
  transform: translateX(-100%);
  transition: transform 0.6s ease;
}

.form-container:hover .title-container h2::after {
  transform: translateX(100%);
}

/* 添加输入框和按钮焦点状态的动效 */
:deep(.el-input__inner:focus),
:deep(.el-textarea__inner:focus) {
  animation: inputPulse 2s infinite;
}

@keyframes inputPulse {
  0% {
    box-shadow: 0 0 0 0 rgba(64, 158, 255, 0.4);
  }
  70% {
    box-shadow: 0 0 0 6px rgba(64, 158, 255, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(64, 158, 255, 0);
  }
}

/* 特小屏幕适配 */
@media (max-width: 360px) {
  .app-wrapper {
    padding: 15px;
  }

  .view-members-btn {
    right: 10px;
    font-size: 12px;
    padding: 3px 8px;
  }

  .title-container h2 {
    margin-right: 70px;
    font-size: 22px;
  }

  .form-container {
    padding: 20px;
  }

}

/* 主题特定样式 */
[data-theme="sakura"] .player-tag {
  background-color: rgba(255, 105, 180, 0.1);
  border-color: rgba(255, 105, 180, 0.2);
  color: #d4317c;
}

[data-theme="sakura"] .player-tag:hover {
  background-color: rgba(255, 105, 180, 0.2);
  box-shadow: 0 4px 12px rgba(255, 105, 180, 0.2);
  text-shadow: 0 0 8px rgba(255, 105, 180, 0.3);
}

[data-theme="cyberpunk"] .player-tag {
  background: rgba(0, 255, 221, 0.1);
  border-color: rgba(246, 24, 246, 0.3);
  color: #00ffd5;
  box-shadow: var(--theme-neon-shadow);
  text-shadow: var(--theme-text-shadow);
}

[data-theme="cyberpunk"] .player-tag:hover {
  background: rgba(0, 255, 221, 0.2);
  box-shadow: 0 0 15px rgba(0, 255, 221, 0.4);
}

/* 暗色模式样式 */
.dark .player-tag {
  background-color: rgba(255, 255, 255, 0.1);
  border-color: var(--theme-border-dark);
  color: var(--theme-text-dark);
}

.dark .player-tag:hover {
  background-color: rgba(255, 255, 255, 0.15);
  border-color: var(--theme-border-dark);
}

/* 在线玩家标签特殊样式 */
.player-tag.el-tag--success {
  background-color: rgba(var(--theme-secondary-rgb), 0.15);
  border-color: var(--theme-secondary);
  color: var(--theme-secondary);
}

.player-tag.el-tag--success:hover {
  background-color: rgba(var(--theme-secondary-rgb), 0.25);
  box-shadow: 0 4px 12px rgba(var(--theme-secondary-rgb), 0.2);
}

/* 修改森林主题下的标签样式，提高可读性 */
[data-theme="forest"] .player-tag {
  background-color: rgba(255, 255, 255, 0.15);
  border-color: rgba(144, 238, 144, 0.3);
  color: #2c5530; /* 深绿色文字 */
  font-weight: 500; /* 加粗文字 */
}

[data-theme="forest"] .player-tag:hover {
  background-color: rgba(255, 255, 255, 0.25);
  border-color: rgba(144, 238, 144, 0.4);
  box-shadow: 0 4px 12px rgba(144, 238, 144, 0.2);
}

/* 暗色模式下的森林主题 */
.dark[data-theme="forest"] .player-tag {
  background-color: rgba(144, 238, 144, 0.15);
  border-color: rgba(144, 238, 144, 0.3);
  color: #90EE90; /* 亮绿色文字 */
}

.dark[data-theme="forest"] .player-tag:hover {
  background-color: rgba(144, 238, 144, 0.25);
  box-shadow: 0 4px 12px rgba(144, 238, 144, 0.15);
}

/* 添加海洋主题特定样式 */
[data-theme="ocean"] .form-container {
  background: rgba(240, 248, 255, 0.95);
  border-color: rgba(100, 181, 246, 0.3);
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(25, 118, 210, 0.15),
  0 2px 8px rgba(25, 118, 210, 0.1);
}

[data-theme="ocean"] .title-container h2 {
  color: #1976D2;
  text-shadow: 0 0 10px rgba(25, 118, 210, 0.2);
}

[data-theme="ocean"] .description {
  color: #0D47A1;
}


[data-theme="ocean"] .server-name {
  color: #1976D2;
  background: rgba(25, 118, 210, 0.1);
}

[data-theme="ocean"] .player-tag {
  background: rgba(3, 169, 244, 0.1);
  border-color: rgba(3, 169, 244, 0.3);
  color: #0277BD;
  font-weight: 500;
}

[data-theme="ocean"] .player-tag:hover {
  background: rgba(3, 169, 244, 0.2);
  box-shadow: 0 4px 12px rgba(3, 169, 244, 0.2);
  text-shadow: 0 0 8px rgba(3, 169, 244, 0.3);
}

[data-theme="ocean"] .header-title {
  color: #1976D2;
}

[data-theme="ocean"] .view-members-btn {
  color: #1976D2;
}

[data-theme="ocean"] .view-members-btn:hover {
  color: #0277BD;
  text-shadow: 0 0 8px rgba(3, 169, 244, 0.3);
}

/* 暗色模式下的海洋主题 */
.dark[data-theme="ocean"] .form-container {
  background: rgba(13, 71, 161, 0.85);
  border-color: rgba(100, 181, 246, 0.2);
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(13, 71, 161, 0.3),
  0 2px 8px rgba(13, 71, 161, 0.2);
}


.dark[data-theme="ocean"] .title-container h2 {
  color: #64B5F6;
  text-shadow: 0 0 10px rgba(100, 181, 246, 0.3);
}

.dark[data-theme="ocean"] .description {
  color: #90CAF9;
}

.dark[data-theme="ocean"] .server-name {
  color: #64B5F6;
  background: rgba(100, 181, 246, 0.15);
}

.dark[data-theme="ocean"] .player-tag {
  background: rgba(3, 169, 244, 0.15);
  border-color: rgba(3, 169, 244, 0.3);
  color: #81D4FA;
}

.dark[data-theme="ocean"] .player-tag:hover {
  background: rgba(3, 169, 244, 0.25);
  box-shadow: 0 4px 12px rgba(3, 169, 244, 0.2);
}

.dark[data-theme="ocean"] .header-title {
  color: #64B5F6;
}

.dark[data-theme="ocean"] .view-members-btn {
  color: #64B5F6;
}

.dark[data-theme="ocean"] .view-members-btn:hover {
  color: #81D4FA;
}

/* 添加极光主题特定样式 */
[data-theme="aurora"] .form-container {
  background: rgba(224, 247, 250, 0.95);
  border-color: rgba(77, 208, 225, 0.3);
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(38, 198, 218, 0.15),
  0 2px 8px rgba(38, 198, 218, 0.1);
}

[data-theme="aurora"] .title-container h2 {
  color: #00838F;
  text-shadow: 0 0 15px rgba(38, 198, 218, 0.3);
}

[data-theme="aurora"] .description {
  color: #006064;
}


[data-theme="aurora"] .server-name {
  color: #00838F;
  background: rgba(38, 198, 218, 0.1);
}

[data-theme="aurora"] .player-tag {
  background: rgba(38, 198, 218, 0.1);
  border-color: rgba(0, 191, 165, 0.3);
  color: #00838F;
  font-weight: 500;
}

[data-theme="aurora"] .player-tag:hover {
  background: rgba(38, 198, 218, 0.2);
  box-shadow: 0 4px 12px rgba(38, 198, 218, 0.2);
  text-shadow: 0 0 8px rgba(38, 198, 218, 0.3);
}

[data-theme="aurora"] .header-title {
  color: #00838F;
}

[data-theme="aurora"] .view-members-btn {
  color: #00838F;
}

[data-theme="aurora"] .view-members-btn:hover {
  color: #00ACC1;
  text-shadow: 0 0 8px rgba(38, 198, 218, 0.3);
}

/* 暗色模式下的极光主题 */
.dark[data-theme="aurora"] .form-container {
  background: rgba(0, 96, 100, 0.85);
  border-color: rgba(77, 208, 225, 0.2);
  backdrop-filter: blur(20px);
  box-shadow: 0 8px 32px rgba(0, 96, 100, 0.3),
  0 2px 8px rgba(0, 96, 100, 0.2);
}


.dark[data-theme="aurora"] .title-container h2 {
  color: #4DD0E1;
  text-shadow: 0 0 15px rgba(77, 208, 225, 0.4);
}

.dark[data-theme="aurora"] .description {
  color: #B2EBF2;
}

.dark[data-theme="aurora"] .server-name {
  color: #4DD0E1;
  background: rgba(77, 208, 225, 0.15);
}

.dark[data-theme="aurora"] .player-tag {
  background: rgba(38, 198, 218, 0.15);
  border-color: rgba(0, 191, 165, 0.3);
  color: #80DEEA;
}

.dark[data-theme="aurora"] .player-tag:hover {
  background: rgba(38, 198, 218, 0.25);
  box-shadow: 0 4px 12px rgba(38, 198, 218, 0.2);
}

.dark[data-theme="aurora"] .header-title {
  color: #4DD0E1;
}

.dark[data-theme="aurora"] .view-members-btn {
  color: #4DD0E1;
}

.dark[data-theme="aurora"] .view-members-btn:hover {
  color: #80DEEA;
}

/* 添加加载状态样式 */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  color: var(--theme-text);
  gap: 10px;
}

.loading-icon {
  font-size: 24px;
  animation: rotate 1s linear infinite;
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 添加内容淡入动画 */
.animate-in {
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 适配暗色模式 */
.dark .loading-state {
  color: var(--theme-text-dark);
}

/* 主题特定样式 */
[data-theme="sakura"] .loading-state {
  color: var(--theme-primary);
}

[data-theme="cyberpunk"] .loading-state {
  color: #00ffd5;
  text-shadow: var(--theme-text-shadow);
}

[data-theme="ocean"] .loading-state {
  color: #1976D2;
}

.dark[data-theme="ocean"] .loading-state {
  color: #64B5F6;
}

[data-theme="aurora"] .loading-state {
  color: #00838F;
}

.dark[data-theme="aurora"] .loading-state {
  color: #4DD0E1;
}

/* 添加查看更多按钮样式 */
.view-more-container {
  display: flex;
  justify-content: center;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid var(--theme-border);
}

.view-more-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  color: var(--theme-primary);
  transition: all 0.3s ease;
  font-family: 'CustomFont', sans-serif;
}

.view-more-btn:hover {
  transform: translateX(4px);
}

/* 添加漂亮的加载和完成动画 */
@keyframes successPulse {
  0% {
    box-shadow: 0 0 0 0 rgba(103, 194, 58, 0.4);
    background-color: #67C23A;
  }
  70% {
    box-shadow: 0 0 0 15px rgba(103, 194, 58, 0);
    background-color: #67C23A;
  }
  100% {
    box-shadow: 0 0 0 0 rgba(103, 194, 58, 0);
    background-color: #67C23A;
  }
}

/* 添加输入框聚焦效果 */
.form-container .animated-form:focus-within {
  transform: scale(1.01);
}

/* 添加标题和描述的进场动画 */
.title-container, .description {
  animation: slideInFromTop 0.6s ease-out;
}

@keyframes slideInFromTop {
  0% {
    opacity: 0;
    transform: translateY(-20px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 优化表单项进场动画 */
.animated-form :deep(.el-form-item) {
  animation: fadeInStaggered 0.5s ease-out backwards;
}

.animated-form :deep(.el-form-item:nth-child(1)) {
  animation-delay: 0.1s;
}

.animated-form :deep(.el-form-item:nth-child(2)) {
  animation-delay: 0.2s;
}

.animated-form :deep(.el-form-item:nth-child(3)) {
  animation-delay: 0.3s;
}

.animated-form :deep(.el-form-item:nth-child(4)) {
  animation-delay: 0.4s;
}

.button-group {
  animation: fadeInStaggered 0.5s ease-out backwards;
  animation-delay: 0.5s;
}

@keyframes fadeInStaggered {
  0% {
    opacity: 0;
    transform: translateX(-20px);
  }
  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

/* 暗色模式输入框样式 */
html.dark :deep(.el-input__inner),
html.dark :deep(.el-textarea__inner) {
  background-color: rgba(30, 30, 40, 0.8) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
  color: rgba(255, 255, 255, 0.9) !important;
  box-shadow: none !important;
}

/* 输入框和文本框的悬停和焦点状态 */
html.dark :deep(.el-input__inner:hover),
html.dark :deep(.el-textarea__inner:hover) {
  border-color: rgba(64, 158, 255, 0.3) !important;
  background-color: rgba(35, 35, 45, 0.8) !important;
}

html.dark :deep(.el-input__inner:focus),
html.dark :deep(.el-textarea__inner:focus) {
  border-color: rgba(64, 158, 255, 0.5) !important;
  background-color: rgba(35, 35, 45, 0.9) !important;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2) !important;
}

/* 输入框和文本框的包装器样式 */
html.dark :deep(.el-input__wrapper),
html.dark :deep(.el-textarea__wrapper) {
  background-color: transparent !important;
  box-shadow: none !important;
}

html.dark :deep(.el-radio__label) {
  color: rgba(255, 255, 255, 0.9);
}

html.dark :deep(.el-form-item__label) {
  color: rgba(255, 255, 255, 0.9);
}

/* 更顺滑的表单聚焦过渡 */
.form-container.focused {
  transform: translateY(-10px);
  transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
</style>
