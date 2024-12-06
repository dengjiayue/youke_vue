<template>
  <div class="app-container">
    <form @submit.prevent="search" inline>
      <label>姓名</label>
      <input v-model="searchParams.Name" placeholder="请输入姓名" />

      <label>电话</label>
      <input v-model="searchParams.PhoneNumber" placeholder="请输入电话" />

      <label>身份证号</label>
      <input v-model="searchParams.IdcardNumber" placeholder="请输入身份证号" />

      <button type="button" @click="search">搜索</button>
    </form>

    <!-- 使用标准HTML table -->
    <table v-if="resultData.Data.length > 0" class="result-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>姓名</th>
          <th>电话</th>
          <th>身份证号</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in resultData.Data" :key="item.Id">
          <td>{{ item.Id }}</td>
          <td>{{ item.Name }}</td>
          <td>{{ item.PhoneNumber }}</td>
          <td>{{ item.IdcardNumber }}</td>
          <td>
            <button @click="openRegisterDialog(item.Id)">登记</button>
            <button @click="goToCreateCustomer(item.Id)">详情</button> <!-- 新增的详情按钮 -->
          </td>
        </tr>
      </tbody>
    </table>

    <div v-else class="empty">暂无数据</div>

    <!-- 弹出框 -->
    <div v-if="showDialog" class="dialog-overlay">
      <div class="dialog-box">
        <h3>登记房间</h3>
        <input v-model="roomNumber" placeholder="请输入房间号" />
        <div class="dialog-buttons">
          <button @click="confirmRegister">确认</button>
          <button @click="cancelRegister">取消</button>
        </div>
      </div>
    </div>

    <!-- 订单按钮 -->
    <button @click="goToOrders" class="order-button">订单</button>
  </div>

  <div>
    <button @click="goToCreateCustomer" class="fixed-button">新增</button>
  </div>
</template>

<script>
import { ref } from 'vue';
import axios from 'axios';
import api_config from './api_config';

export default {
  methods: {
    goToCreateCustomer(id) {
      // 基本的跳转 URL
      let url = '/create_customer.html';
  
      // 如果有传入 id，则拼接到 URL 中
      if (id) {
        url += `?id=${id}`;
      }

      // 跳转到指定 URL
      window.location.href = url;
    },
    goToOrders() {
      // 跳转到订单页面
      window.location.href = '/order.html';
    }
  },

  setup() {
    const searchParams = ref({
      Name: '',
      PhoneNumber: '',
      IdcardNumber: '',
    });

    const resultData = ref({
      Count: 0,
      Data: [],
    });

    const showDialog = ref(false);
    const selectedCustomerId = ref(null);
    const roomNumber = ref('');

    const search = async () => {
      try {
        const response = await axios.post(`${api_config.apiBaseUrl}/SelectCostomerSimple`, searchParams.value);
        if (response.data.Code === 0) {
          resultData.value = response.data.Data;
        } else {
          resultData.value = { Count: 0, Data: [] };
        }
      } catch (error) {
        console.error('请求失败:', error);
        resultData.value = { Count: 0, Data: [] };
      }
    };

    const openRegisterDialog = (customerId) => {
      selectedCustomerId.value = customerId;
      roomNumber.value = '';
      showDialog.value = true;
    };

    const confirmRegister = async () => {
      if (!roomNumber.value) {
        alert("请输入房间号");
        return;
      }
      try {
        const response = await axios.post(`${api_config.apiBaseUrl}/CreatOrder`, {
          CustomerId: selectedCustomerId.value,
          RoomNumber: roomNumber.value
        });
        alert(response.data.Msg); // 提示返回的消息
        if (response.data.Code === 0) {
          showDialog.value = false;
        }
      } catch (error) {
        console.error('登记请求失败:', error);
        alert('登记失败');
      }
    };

    const cancelRegister = () => {
      showDialog.value = false;
    };

    return {
      searchParams,
      resultData,
      search,
      showDialog,
      roomNumber,
      openRegisterDialog,
      confirmRegister,
      cancelRegister
    };
  },
};
</script>

<style scoped>
.app-container {
  padding: 20px;
}

.result-table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
}

.result-table th,
.result-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.empty {
  margin-top: 20px;
  color: #888;
}

.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.dialog-box {
  background: #fff;
  padding: 20px;
  border-radius: 5px;
  width: 300px;
}

.dialog-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}

.dialog-buttons button {
  padding: 5px 10px;
}

.fixed-button {
  position: fixed; /* 固定位置 */
  bottom: 20px; /* 距离底部20像素 */
  left: 50%; /* 左侧距离50% */
  transform: translateX(-50%); /* 水平居中对齐 */
  padding: 10px 20px; /* 按钮内边距 */
  background-color: #007bff; /* 按钮背景色 */
  color: white; /* 按钮文字颜色 */
  border: none; /* 去掉边框 */
  border-radius: 5px; /* 圆角 */
  cursor: pointer; /* 鼠标样式 */
  z-index: 1000; /* 确保按钮在最上层 */
}

.order-button {
  position: absolute; /* 绝对定位 */
  top: 20px; /* 距离顶部20像素 */
  right: 20px; /* 距离右边20像素 */
  padding: 10px 15px; /* 按钮内边距 */
  background-color: #28a745; /* 按钮背景色 */
  color: white; /* 按钮文字颜色 */
  border: none; /* 去掉边框 */
  border-radius: 5px; /* 圆角 */
  cursor: pointer; /* 鼠标样式 */
  z-index: 1000; /* 确保按钮在最上层 */
}
</style>
