<template>
    <div class="app-container">
      <h1>订单列表</h1>
  
      <!-- 当前日期按钮 -->
      <div class="date-button">
        <button @click="showDatePicker = true">{{ formatDate(currentDate) }}</button>
      </div>
  
      <table v-if="orders.length > 0" class="order-table">
        <thead>
          <tr>
            <th>操作</th>
            <th>ID</th>
            <th>客户姓名</th>
            <th>电话</th>
            <th>房间号</th>
            <th>日期</th>
            <th>创建时间</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="order in orders" :key="order.Id">
            <td>
              <button @click="goToCreateCustomer(order.CustomerId)">详情</button>
            </td>
            <td>{{ order.Id }}</td>
            <td>{{ order.CustomerName }}</td>
            <td>{{ order.PhoneNumber }}</td>
            <td>{{ order.RoomNumber }}</td>
            <td>{{ formatDate(order.Ymd) }}</td>
            <td>{{ formatDate(order.CreatedAt) }}</td>
          </tr>
        </tbody>
      </table>
  
      <div v-else class="empty">暂无订单</div>
  
      <div class="fixed-buttons">
        <button @click="changeDate(-1)">前一天</button>
        <button @click="changeDate(1)">后一天</button>
      </div>
  
      <!-- 日期选择器 -->
      <div v-if="showDatePicker" class="date-picker-overlay">
        <div class="date-picker">
          <h3>选择日期</h3>
          <input type="date" v-model="selectedDate" />
          <div class="date-picker-buttons">
            <button @click="confirmDate">确认</button>
            <button @click="cancelDate">取消</button>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  import api_config from '../api_config';
  
  export default {
    setup() {
      const orders = ref([]);
      const currentDate = ref(new Date());
      const showDatePicker = ref(false);
      const selectedDate = ref(currentDate.value.toISOString().split('T')[0]); // 默认选中当前日期
  
      const fetchOrders = async () => {
        const ymd = currentDate.value.toISOString().split('T')[0];
        try {
          const response = await axios.post(`${api_config.apiBaseUrl}/SelectOrderByYmd`, {
            Ymd: ymd
          });
          if (response.data.Code === 0) {
            orders.value = response.data.Data;
          } else {
            orders.value = [];
            console.error(response.data.Msg);
          }
        } catch (error) {
          console.error('请求失败:', error);
          orders.value = [];
        }
      };
  
      const goToCreateCustomer = (id) => {
        let url = '/create_customer.html';
        if (id) {
          url += `?id=${id}`;
        }
        window.location.href = url;
      };
  
      const formatDate = (dateString) => {
        const options = { year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit', second: '2-digit', timeZoneName: 'short' };
        return new Date(dateString).toLocaleString(undefined, options);
      };
  
      const changeDate = (offset) => {
        currentDate.value.setDate(currentDate.value.getDate() + offset);
        fetchOrders();
      };
  
      const confirmDate = () => {
        currentDate.value = new Date(selectedDate.value); // 更新当前日期
        fetchOrders(); // 重新请求数据
        showDatePicker.value = false; // 关闭日期选择器
      };
  
      const cancelDate = () => {
        showDatePicker.value = false; // 关闭日期选择器
      };
  
      onMounted(() => {
        fetchOrders();
      });
  
      return {
        orders,
        goToCreateCustomer,
        formatDate,
        changeDate,
        showDatePicker,
        selectedDate,
        confirmDate,
        cancelDate,
        currentDate,
      };
    },
  };
  </script>
  
  <style scoped>
  .app-container {
    padding: 20px;
  }
  
  .date-button {
    position: absolute; /* 绝对定位 */
    top: 20px; /* 距离顶部20像素 */
    right: 20px; /* 距离右侧20像素 */
  }
  
  .order-table {
    width: 100%;
    margin-top: 20px;
    border-collapse: collapse;
  }
  
  .order-table th,
  .order-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  
  .empty {
    margin-top: 20px;
    color: #888;
  }
  
  .fixed-buttons {
    position: fixed;
    left: 20px;
    bottom: 20px;
  }
  
  .fixed-buttons button {
    margin-right: 10px;
  }
  
  .date-picker-overlay {
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
  
  .date-picker {
    background: #fff;
    padding: 20px;
    border-radius: 5px;
    width: 300px;
  }
  
  .date-picker-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
  }
  </style>
  