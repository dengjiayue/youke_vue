<template>
    <div class="app-container">
      <el-form :model="searchParams" inline>
        <el-form-item label="姓名">
          <el-input v-model="searchParams.Name" placeholder="请输入姓名" />
        </el-form-item>
        <el-form-item label="电话">
          <el-input v-model="searchParams.PhoneNumber" placeholder="请输入电话" />
        </el-form-item>
        <el-form-item label="身份证号">
          <el-input v-model="searchParams.IdcardNumber" placeholder="请输入身份证号" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="search">搜索</el-button>
        </el-form-item>
      </el-form>
  
      <el-table
        v-if="resultData.Data.length > 0"
        :data="resultData.Data"
        height="400"
        style="width: 100%; margin-top: 20px"
      >
        <el-table-column prop="Id" label="ID" width="80" />
        <el-table-column prop="Name" label="姓名" />
        <el-table-column prop="PhoneNumber" label="电话" />
        <el-table-column prop="IdcardNumber" label="身份证号" />
      </el-table>
  
      <el-empty v-else description="暂无数据" />
    </div>
  </template>
  
  <script>
  import { ref } from 'vue';
  import axios from 'axios';
  
  export default {
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
  
      const search = async () => {
        try {
          const response = await axios.post('http://localhost:11000/SelectCostomerSimple', searchParams.value);
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
  
      return {
        searchParams,
        resultData,
        search,
      };
    },
  };
  </script>
  
  <style scoped>
  .app-container {
    padding: 20px;
  }
  </style>
  