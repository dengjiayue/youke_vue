<template>
    <div class="registration">
      <h2>登记表单</h2>
      <form @submit.prevent="submitForm">
        <div>
          <label for="child_id_number">被监护人身份证号:</label>
          <input type="text" v-model="form.child_id_number" id="child_id_number" />
        </div>
        <div>
          <label for="room_number">房间号:<span style="color: red;">*</span></label>
          <input type="text" v-model="form.room_number" id="room_number" required />
        </div>
        <div>
          <label for="name">姓名:<span style="color: red;">*</span></label>
          <input type="text" v-model="form.name" id="name" required />
          <button type="button" @click="copyToClipboard(form.name)">复制</button>
        </div>
        <div>
          <label for="phone_number">电话:<span style="color: red;">*</span></label>
          <input type="tel" v-model="form.phone_number" id="phone_number" required />
          <button type="button" @click="copyToClipboard(form.phone_number)">复制</button>
        </div>
        <div>
          <label for="idcard_number">身份证号:<span style="color: red;">*</span></label>
          <input type="text" v-model="form.idcard_number" id="idcard_number" required />
          <button type="button" @click="copyToClipboard(form.idcard_number)">复制</button>
        </div>
        <div>
          <label for="address">地址:<span style="color: red;">*</span></label>
          <input type="text" v-model="form.address" id="address" required />
          <button type="button" @click="copyToClipboard(form.address)">复制</button>
        </div>
        <div>
          <label for="guardian_id">监护人ID:</label>
          <input type="number" v-model="form.guardian_id" id="guardian_id" />
          <button type="button" @click="getGuardianDetails(form.guardian_id)">详情</button>
        </div>
        <div>
          <label for="idcard_img">身份证:<span style="color: red;">*</span></label>
          <input type="file" @change="handleIdCardChange" id="idcard_img" />
          <div class="image-preview">
            <img v-if="preview.idcard_img" :src="preview.idcard_img" alt="身份证预览" class="preview-img" />
            <button v-if="preview.idcard_img" type="button" @click="downloadImage(preview.idcard_img)" class="download-button">下载</button>
          </div>
        </div>
        <div>
          <label for="face_img">头像照:<span style="color: red;">*</span></label>
          <input type="file" @change="uploadFaceImage" id="face_img" />
          <div class="image-preview">
            <img v-if="preview.face_img" :src="preview.face_img" alt="头像预览" class="preview-img" />
            <button v-if="preview.face_img" type="button" @click="downloadImage(preview.face_img)" class="download-button">下载</button>
          </div>
        </div>
        <div class="button-group">
          <button type="button" @click="bindGuardian">绑定监护人</button>
          <button type="submit">提交</button>
        </div>
      </form>
      <div v-if="message" class="message">{{ message }}</div>
      <div v-if="isAdult !== null" :style="{ color: isAdult ? 'black' : 'red' }">
        顾客状态: {{ isAdult ? '成年' : '未成年' }}
      </div>
    </div>
  </template>
  
  <script>
  import api_config from '../api_config';
  
  export default {
    data() {
      return {
        form: {
          child_id_number: '',
          room_number: '',
          name: '',
          phone_number: '',
          face_img: '',
          idcard_img: '',
          idcard_number: '',
          address: '',
          guardian_id: 0,
        },
        message: '',
        isAdult: null,
        preview: {
          face_img: null,
          idcard_img: null,
        },
      };
    },
    mounted() {
      const urlParams = new URLSearchParams(window.location.search);
      const id = urlParams.get('id');
  
      if (id) {
        this.getGuardianDetails(id);
      }
    },
    methods: {
      async uploadFaceImage(event) {
        const file = event.target.files[0];
        const idcardNumber = this.form.idcard_number;
  
        if (file && idcardNumber) {
          const formData = new FormData();
          formData.append('face_img', file);
          formData.append('idcard_number', idcardNumber);
  
          try {
            const response = await fetch(`${api_config.apiBaseUrl}/UploadFaceImg`, {
              method: 'POST',
              body: formData,
            });
            const result = await response.json();
  
            if (result.Code === 0) {
              this.form.face_img = result.Data.FaceImg;
              this.preview.face_img = `${result.Data.FaceImg}?t=${new Date().getTime()}`;
            } else {
              console.error('上传失败:', result.Msg);
            }
          } catch (error) {
            console.error('上传头像照失败:', error);
          }
        } else {
          console.warn("请先填写身份证号再选择头像文件");
        }
      },
  
      async handleIdCardChange(event) {
        const file = event.target.files[0];
        if (file) {
          const reader = new FileReader();
          reader.onload = (e) => {
            this.preview.idcard_img = e.target.result;
          };
          reader.readAsDataURL(file);
  
          const base64 = await this.convertToBase64(file);
          await this.recognizeIdCard(base64);
        }
      },
  
      convertToBase64(file) {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onloadend = () => resolve(reader.result.split(',')[1]);
          reader.onerror = reject;
          reader.readAsDataURL(file);
        });
      },
  
      async recognizeIdCard(base64) {
        try {
          const response = await fetch(`${api_config.apiBaseUrl}/IdCardRecognition`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            body: JSON.stringify({ IdCardBase64: base64 }),
          });
          const result = await response.json();
  
          if (result.Code === 0) {
            const data = result.Data;
  
            this.form.name = data.Name || '';
            this.form.address = data.Address || '';
            this.form.idcard_number = data.IdNum || '';
            this.form.idcard_img = data.IdcardImg || '';
            this.form.face_img = data.HeadImg || '';
            this.isAdult = data.IsAdult;
  
            this.preview.idcard_img = data.IdcardImg || '';
            this.preview.face_img = data.HeadImg || '';
          } else {
            console.error('请求失败:', result.Msg);
          }
        } catch (error) {
          console.error('身份证识别失败:', error);
        }
      },
  
      async submitForm() {
        if (!this.form.face_img || !this.form.idcard_img) {
          this.message = '请确保身份证和头像链接有效！';
          return;
        }
  
        const formData = new FormData();
        for (const key in this.form) {
          formData.append(key, this.form[key]);
        }
  
        try {
          const response = await fetch(`${api_config.apiBaseUrl}/CreatOrderAndUpdateCostomer`, {
            method: 'POST',
            body: formData,
          });
          const result = await response.json();
          this.message = result.Msg;
  
          setTimeout(() => {
            this.message = '';
          }, 3000);
        } catch (error) {
          console.error('提交失败:', error);
        }
      },
  
      bindGuardian() {
        const idCardNumber = this.form.idcard_number;
        this.form.room_number = '';
        this.form.name = '';
        this.form.phone_number = '';
        this.form.face_img = '';
        this.form.idcard_img = '';
        this.form.idcard_number = '';
        this.form.address = '';
        this.form.guardian_id = '';
        this.form.child_id_number = idCardNumber;
        this.preview.face_img = '';
        this.preview.idcard_img = '';
      },
  
      async getGuardianDetails(customer_id) {
        const id = Number(customer_id);
        if (id) {
          try {
            const response = await fetch(`${api_config.apiBaseUrl}/SelectCostomerById`, {
              method: 'POST',
              headers: {
                'Content-Type': 'application/json',
              },
              body: JSON.stringify({ Id: id }),
            });
            const result = await response.json();
  
            if (result.Code === 0) {
              const data = result.Data;
              this.form.name = data.Name || '';
              this.form.phone_number = data.PhoneNumber || '';
              this.form.idcard_number = data.IdcardNumber || '';
              this.form.address = data.Address || '';
              this.preview.face_img = data.FaceImg || '';
              this.preview.idcard_img = data.IdcardImg || '';
              this.form.guardian_id = data.GuardianId || null;
            } else {
              console.error('请求失败:', result.Msg);
            }
          } catch (error) {
            console.error('获取监护人详情失败:', error);
          }
        }
      },
  
      copyToClipboard(text) {
        navigator.clipboard.writeText(text)
          .then(() => {
            this.message = '复制成功！';
            setTimeout(() => {
              this.message = '';
            }, 2000);
          })
          .catch(err => {
            console.error('复制失败:', err);
          });
      },
  
      downloadImage(url) {
        const link = document.createElement('a');
        link.href = url;
        link.download = url.split('/').pop();
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      },
    }
  };
  </script>
  
  <style scoped>
  .registration {
    max-width: 400px;
    margin: auto;
  }
  .message {
    margin-top: 10px;
    color: green;
  }
  .image-preview {
    display: flex;
    align-items: center;
    margin-top: 10px;
  }
  .preview-img {
    max-width: 150px;
    max-height: 150px;
    object-fit: contain;
    margin-right: 10px; /* 添加右边距以分隔图片和按钮 */
  }
  .download-button {
    margin-left: 10px; /* 按钮与图片之间的间距 */
  }
  .button-group {
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
  }
  </style>
  