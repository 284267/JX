<template>
  <div class="login-container">
    <t-card class="login-card">
      <h2 class="login-title">Welcome Back</h2>
      <t-loading :loading="loading">
        <t-form :data="loginForm" :label-width="0" :rules="rules" ref="loginFormRef" @submit="onSubmit">
          <t-form-item name="username">
            <t-input v-model="loginForm.username" placeholder="Email Address or Phone Number"></t-input>
          </t-form-item>
          <t-form-item name="password">
            <t-input v-model="loginForm.password" type="password" autocomplete="on" placeholder="password"></t-input>
          </t-form-item>
          <t-form-item>
            <t-button theme="primary" type="submit" size="large" class="login-button">Continue</t-button>
          </t-form-item>
        </t-form>
      </t-loading>
      <div class="signup-prompt">
         <t-link :underline="false" href="/admin/#/register"></t-link>
      </div>
    </t-card>
  </div>
</template>

<script setup lang="ts">
import { FormInstanceFunctions, FormProps, FormRule } from 'tdesign-vue-next';
import { computed, onMounted, reactive, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';

import LogoOpenai from '@/assets/openai-logo.svg';
import { ChatgptTokenTutorialUrl } from '@/constants/index';
import { useUserStore } from '@/store';

const userStore = useUserStore();
const loading = ref(false);
const route = useRoute();
const router = useRouter();
const cfg = ref({ enable_plugin: false, notice: '' });
const loginForm = reactive({
  username: '',
  password: '',
  chatgpt_token: undefined,
  invite_token: undefined,
  invite_id: undefined,
});

onMounted(async () => {
  // await getFeCfg();
});

const rules: Record<string, FormRule[]> = {
  username: [{ required: true, message: 'Please enter your username', trigger: 'blur' }],
  password: [{ required: true, message: 'Please enter your password', trigger: 'blur' }],
  chatgpt_token: [
    { required: true, message: '请输入 Access Token 或 Session Token 或 Refresh Token', trigger: 'blur' },
  ],
};

const loginFormRef = ref<FormInstanceFunctions>(null);

const IsRegister = computed(() => {
  return !route.path.endsWith('/login');
});

const loginType = computed(() => {
  if (route.path.endsWith('/register')) {
    return 'register';
  }
  if (route.path.endsWith('/invite_register')) {
    return 'invite_register';
  }
  return 'login';
});

const onSubmit: FormProps['onSubmit'] = async ({ validateResult, firstError }) => {
  if (validateResult === true) {
    loading.value = true;
    let url;

    switch (loginType.value) {
      case 'register':
        url = '/0x/user/register';
        break;
      default:
        url = '/0x/user/login';
    }
    if (loginType.value === 'invite_register') {
      const { hash } = window.location;
      const paramsString = hash.split('?')[1];
      const params = new URLSearchParams(paramsString);
      loginForm.invite_token = params.get('invite_token');
      loginForm.invite_id = params.get('id');
    }

    const data = await userStore.login(url, loginForm);
    if (data.admin_token && data.is_admin) {
      router.push({ name: 'User' });
    } else if (data.admin_token) {
      return router.push({ name: 'LoginChatgpt' });
    }

    loading.value = false;
  } else {
    console.error('表单引用未定义', firstError);
  }
};

const getFeCfg = async () => {
  const response = await fetch('/api/fe-cfg');
  const data = await response.json();
  Object.assign(cfg.value, { ...data.data });
  return data;
};

const goFree = async () => {
  loading.value = true;
  const data = await userStore.login('/0x/user/login-free', {});
  if (data.admin_token) {
    return router.push({ name: 'LoginChatgpt' });
  }

  loading.value = false;
};
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f7f7f8; /* 背景颜色，白色略带灰色 */
}

.login-card {
  width: 400px;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); /* 卡片阴影 */
  text-align: center;
  background-color: white; /* 背景颜色 */
}

.login-title {
  font-size: 24px;
  font-weight: bold;
  color: #343541; /* 标题颜色，黑灰色 */
  margin-bottom: 20px;
}

.login-button {
  width: 100%;
  height: 45px;
  background-color: #10a37f; /* 按钮颜色，绿色 */
  border-color: #10a37f;
  color: white;
  font-size: 16px;
  font-weight: bold;
  margin-top: 20px;
}

.login-button:hover {
  background-color: #0e8a6f; /* 按钮悬停颜色 */
  border-color: #0e8a6f;
}

.signup-prompt {
  margin-top: 15px;
  font-size: 14px;
  color: #343541; /* 链接颜色，黑灰色 */
}
</style>
