<template>
  <div class="services-form">
    <!-- Режим просмотра данных -->
    <div v-if="!editMode">
      <h3 style="margin-top: 20px;">Добавленные услуги:</h3>
      <a-list bordered>
        <a-list-item v-for="(service, index) in services" :key="index">
          <div style="flex-grow: 1;">
            {{ service.category }} - {{ service.cost }} ₽
          </div>
        </a-list-item>
      </a-list>

      <h3 style="margin-top: 20px;">Рабочее время:</h3>
      <a-list bordered>
        <a-list-item>
          <div style="flex-grow: 1;">
            Начало: {{ startTime }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Конец: {{ endTime }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Рабочие дни: {{ workingDays.join(', ') }}
          </div>
        </a-list-item>
      </a-list>

      <div class="button-container">
        <a-button type="primary" @click="toggleEditMode">
          Редактировать информацию
        </a-button>
      </div>
    </div>

    <!-- Режим редактирования данных -->
    <div v-else>
      <h3 style="margin-top: 20px;">Добавленные услуги:</h3>
      <a-list bordered>
        <a-list-item v-for="(service, index) in services" :key="index">
          <div style="flex-grow: 1;">
            {{ service.category }} - {{ service.cost }} ₽
          </div>
          <a-button type="link" @click="removeService(index)" style="color: red;">
            Удалить
          </a-button>
        </a-list-item>
      </a-list>

      <div class="button-container">
        <a-button type="primary" @click="showModal" class="mb-4">
          Создать услугу
        </a-button>

        <!-- Модальное окно для добавления услуги -->
        <a-modal
          v-model:visible="isModalVisible"
          title="Создание услуги"
          @ok="addService"
          @cancel="handleCancel"
        >
          <a-form layout="vertical">
            <a-form-item label="Категория услуги">
              <a-select v-model="selectedCategory" placeholder="Выберите категорию" @change="onCategoryChange">
                <a-select-option v-for="category in categories" :key="category" :value="category">
                  {{ category }}
                </a-select-option>
              </a-select>
            </a-form-item>

            <a-form-item label="Стоимость">
              <a-input-number
                v-model="cost"
                placeholder="Введите стоимость"
                style="width: 100%;"
                :min="0"
                :formatter="value => `${value}`"
                @change="onCostChange"
              />
            </a-form-item>
          </a-form>
        </a-modal>
      </div>

      <h3 style="margin-top: 20px;">Настройка рабочего времени</h3>
      <a-form layout="vertical">
        <a-form-item label="Начало">
          <a-time-picker v-model="startTime" readOnly={true} @change="onStartTimeChange" format="HH:mm" placeholder=""/>
        </a-form-item>
        <a-form-item label="Конец">
          <a-time-picker v-model="endTime" @change="onEndTimeChange" format="HH:mm" placeholder=""/>
        </a-form-item>
        <a-form-item label="Рабочие дни">
          <a-checkbox-group v-model="workingDays" @change="onWorkingDaysChange">
            <a-row>
              <a-col v-for="day in days" :key="day" span="8">
                <a-checkbox :value="day">{{ day }}</a-checkbox>
              </a-col>
            </a-row>
          </a-checkbox-group>
        </a-form-item>
      </a-form>

      <a-button type="primary" @click="saveInfo" style="margin-top: 20px;">
        Сохранить информацию
      </a-button>
      <a-button @click="toggleEditMode" style="margin-top: 20px; margin-left: 10px;">
        Отмена
      </a-button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'ServicesForm',
  data() {
    return {
      categories: ['Услуга 1', 'Услуга 2', 'Услуга 3'],
      selectedCategory: null,
      cost: null,
      startTime: null,
      endTime: null,
      days: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
      workingDays: [],
      services: [],
      userId: null, // примерный ID пользователя
      editMode: false,
      isModalVisible: false,
      base_url: "https://e270-188-243-183-39.ngrok-free.app"
    };
  },
  computed: {
    formattedStartTime() {
      return this.startTime ? this.formatTime(this.startTime) : 'Не установлено';
    },
    formattedEndTime() {
      return this.endTime ? this.formatTime(this.endTime) : 'Не установлено';
    },
  },
  mounted() {
    const script = document.createElement('script');
    script.src = 'https://telegram.org/js/telegram-web-app.js';
    script.onload = () => {
      window.Telegram.WebApp.ready();

      const user = window.Telegram.WebApp.initDataUnsafe.user;
      if (user && user.id) {
        this.userId = user.id;
        this.loadInitialData();
      }
    };
    document.head.appendChild(script);
  },
  methods: {
    showModal() {
      this.isModalVisible = true;
    },
    handleCancel() {
      this.isModalVisible = false;
      this.resetForm();
    },
    loadInitialData() {
      axios.get(`${this.base_url}/api/v1/orders/${this.userId}/`, { 'headers': { 'ngrok-skip-browser-warning': "oke" } })
        .then(response => {
          const data = response.data;
          if (data.tasks.length === 0){
            this.editMode = true;
          }
          else{
            this.services = data.tasks || [];
            this.startTime = data.time_start ? this.formatTime(data.time_start) : null;
            this.endTime = data.time_end ? this.formatTime(data.time_end) : null;
            this.workingDays = data.work_days || [];
          }
          
        })
        .catch(error => {
            this.editMode = 1;
            alert(error);
          console.error('Ошибка при загрузке данных:', error);
        });
    },
    formatTime(isoString) {
      const date = new Date(isoString);
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      return `${hours}:${minutes}`;
    },
    toggleEditMode() {
      this.editMode = !this.editMode;
    },
    removeService(index) {
      this.services.splice(index, 1);
    },
    resetForm(){
        this.selectedCategory = null;
        this.cost = null;

    },
    addService() {
      if (this.selectedCategory && this.cost !== null) {
        this.services.push({
          category: this.selectedCategory,
          cost: this.cost,
        });

        this.isModalVisible = false;

        // Сброс формы после добавления услуги
        this.resetForm();
      } else {
        alert('Пожалуйста, выберите категорию и укажите стоимость');
      }
    },
    onCategoryChange(value) {
      this.selectedCategory = value;
      console.log('Выбрана категория:', this.selectedCategory);
    },
    onCostChange(value) {
      this.cost = value;
      console.log('Стоимость изменена:', this.cost);
    },
    onStartTimeChange(value) {
      this.startTime = value;
      console.log('Начало рабочего времени:', this.startTime);
    },
    onEndTimeChange(value) {
      this.endTime = value;
      console.log('Конец рабочего времени:', this.endTime);
    },
    onWorkingDaysChange(value) {
      this.workingDays = value;
      console.log('Выбранные рабочие дни:', this.workingDays);
    },
    async saveInfo() {

      if (!this.startTime || !this.endTime || this.workingDays.length === 0) {
        alert('Пожалуйста, укажите рабочее время и выберите рабочие дни');
        return;
      }

      const dataToSend = {
        tasks: this.services,
        time_start: this.startTime,
        time_end: this.endTime,
        work_days: this.workingDays,
        user_id: this.userId.toString(),
      };

      this.startTime = this.formatTime(dataToSend.time_start);
      this.endTime = this.formatTime(dataToSend.time_end);

      console.log(dataToSend);

      try {
        await axios.get(`${this.base_url}/api/v1/orders/${this.userId}/`);
        await axios.put(`${this.base_url}/api/v1/orders/${this.userId}`, dataToSend);
        alert('Информация успешно сохранена');
        this.toggleEditMode(); // Вернуться в режим просмотра после сохранения
      } catch (error) {
        await axios.post(`${this.base_url}/api/v1/orders/`, dataToSend);
        this.editMode = 0;
        console.error('Информация успешно сохранена', error);

      }
    },
  },
};
</script>

<style scoped>
.services-form {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #d9d9d9;
  border-radius: 8px;
  background-color: #fafafa;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
.button-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  margin-bottom: 16px;
}
h2, h3 {
  color: #333;
}
</style>
