<template>
  <div class="container">
    <t-dialog :visible="true" header="选择ChatGPTPlus" :cancel-btn="null" :confirm-btn="null" :on-close="onClose" class="dialog-wrapper">
      <t-list class="list-block" split>
        <t-loading :loading="tableLoading">
          <t-list-item v-for="item in tableData" :key="item.id" @click="onSelect(item.id)">
            <t-list-item-meta>
              <template #description>
                <t-space>
                  <t-tag theme="primary" variant="outline">{{ item.plan_type }}</t-tag>
                  <span>{{ item.chatgpt_flag }}</span>
                </t-space>
              </template>
            </t-list-item-meta>
          </t-list-item>
        </t-loading>
      </t-list>
    </t-dialog>
  </div>
</template>

<script setup lang="ts">
import Cookies from 'js-cookie';
import { MessagePlugin } from 'tdesign-vue-next';
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';

import RequestApi from '@/api/request';

const tableLoading = ref(false);
const router = useRouter();

interface TableData {
  id: number;
  chatgpt_flag: string;
  plan_type: string;
}
const tableData = ref<TableData[]>([]);

onMounted(async () => {
  await getUserChatGPTAccountList();
});

const getUserChatGPTAccountList = async () => {
  // 获取 用户 ChatGPT 账号列表
  tableLoading.value = true;

  const response = await RequestApi('/0x/user/chatgpt-list');

  let data;
  try {
    data = await response.json();
  } catch (error) {
    MessagePlugin.error('登录请求失败，请稍后再试。');
    tableLoading.value = false;
    return;
  }

  tableData.value = data.results;
  tableLoading.value = false;
};

const onClose = () => {
  router.push({ name: 'login' });
};
const onSelect = async (chatgptId: number) => {
  console.log(chatgptId);
  tableLoading.value = true;
  const response = await RequestApi('/0x/chatgpt/login', 'POST', { chatgpt_id: chatgptId });
  tableLoading.value = false;

  let data;
  try {
    data = await response.json();
  } catch (error) {
    MessagePlugin.error('登录请求失败，请稍后再试。');
    return;
  }

  if (response.status !== 200) {
    MessagePlugin.error(data.message || '无法登录，请检查您的凭证后再试。');
  } else {
    Cookies.set('user-gateway-token', data['user-gateway-token'], { expires: 7 });
    MessagePlugin.success('登录成功');
    window.location.href = '/'; // 跳转到首页
  }
};
</script>

<style lang="less" scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;  // Full height to center the dialog vertically
  background: linear-gradient(45deg, #f2ebff, #f2f7f2);  // Gradient background updated to be more subtle, matching the image
  animation: gradientAnimation 10s infinite alternate;
}

@keyframes gradientAnimation {
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

.dialog-wrapper {
  max-width: 360px;  // Set a fixed width similar to the original design
  width: 100%;
  border-radius: 15px;  // Rounded corners for the dialog
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);  // Shadow for depth
  background: #ffffff;  // White background for the dialog
  padding: 30px;  // Padding for better layout
}

.list-block {
  border: none;  // Remove border for a clean look
  border-bottom: none;

  .t-list-item {
    transition: background 0.3s ease, box-shadow 0.3s ease;

    &:hover {
      background: #f5f7fa;  // Light hover background
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);  // Subtle shadow
      cursor: pointer;
    }
  }

  .t-list-item-meta {
    font-family: Arial, sans-serif;  // Use concise font style
    font-weight: 400;  // Lighter font weight
  }

  .t-tag {
    font-size: 16px;  // Font size for tags
    color: #008080;  // Changed tag color to match the teal color in the provided image
    border-color: #008080;  // Outline color
    border-radius: 4px;  // Rounded corners for tags
  }

  .t-space {
    margin: 20px 0;  // Comfortable spacing
  }
}

.t-dialog {
  .t-dialog__header {
    font-size: 28px;  // Title font size
    text-align: center;  // Centered title
    font-family: Arial, sans-serif;  // Concise font
    font-weight: bold;  // Prominent title
    color: #333;  // Set title color to dark gray for readability
  }

  .t-dialog__body {
    padding: 30px;  // Spacious layout
  }
}

.t-button {
  width: 100%;  // Full-width button
  background-color: #008080;  // Main button color updated to match the image
  color: #fff;  // White text
  font-size: 18px;  // Button text size
  padding: 15px;  // Padding for better clickability
  border: none;  // Clean look
  border-radius: 10px;  // Softer button look
  transition: background-color 0.3s ease, box-shadow 0.3s ease;  // Smooth transition

  &:hover {
    background-color: #006666;  // Darker teal hover effect
    box-shadow: 0 0 20px #006666;  // Hover shadow
  }
}
</style>
