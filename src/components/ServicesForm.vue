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
            Начало: {{ formattedStartTime }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Конец: {{ formattedEndTime }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Рабочие дни: {{ workingDays.join(', ') }}
          </div>
        </a-list-item>
      </a-list>

      <div class="button-container">
        <a-button type="primary" size="large" @click="toggleEditMode">
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
          <a-button type="link" size="large" @click="removeService(index)" style="color: red;">
            Удалить
          </a-button>
        </a-list-item>
      </a-list>

      <div class="button-container">
        <a-button type="primary" size="large" @click="showServiceModal" class="mb-4">
          Добавить услугу
        </a-button>

        <!-- Модальное окно для добавления услуги -->
        <a-modal
          v-model:visible="isServiceModalVisible"
          title="Создание услуги"
          @ok="addService"
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
          <template #footer>
            <a-button type="primary" size="large" @click="addService" style="width: 100%;">
              Добавить
            </a-button>
          </template>
        </a-modal>
      </div>

      <h3 style="margin-top: 20px;">Настройка рабочего времени</h3>
      <a-form layout="vertical">
        <a-form-item label="Начало">
          <a-select v-model="startTime" @change="onStartTimeChange" placeholder="Выберите время начала">
            <a-select-option
              v-for="time in timeOptions"
              :key="time"
              :value="time"
            >
              {{ time }}
            </a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item label="Конец">
          <a-select v-model="endTime" @change="onEndTimeChange" placeholder="Выберите время конца">
            <a-select-option
              v-for="time in timeOptionsReverse"
              :key="time"
              :value="time"
            >
              {{ time }}
            </a-select-option>
          </a-select>
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

      <div class="button-container" style="flex-direction: column;">
        <a-button type="primary" size="large" @click="saveInfo" style="width: 100%; margin-bottom: 10px;">
          Сохранить информацию
        </a-button>
        <a-button size="large" @click="toggleEditMode" style="width: 100%;">
          Отмена
        </a-button>
      </div>
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
      isServiceModalVisible: false,
      days: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
      workingDays: [],
      services: [],
      userId: null,
      editMode: false,
      base_url: "https://89aa-2a00-f940-2-4-2-00-1b9c.ngrok-free.app",
    };
  },
  computed: {
    formattedStartTime() {
      return this.startTime ? this.startTime : '';
    },
    formattedEndTime() {
      return this.endTime ? this.endTime : '';
    },
    timeOptions() {
      const options = [];
      for (let hour = 0; hour < 24; hour++) {
        const formattedHour = hour < 10 ? `0${hour}` : hour;
        options.push(`${formattedHour}:00`);
      }
      return options;
    },
    timeOptionsReverse() {
      const options = [];
      for (let hour = 0; hour < 24; hour++) {
        const formattedHour = hour < 10 ? `0${hour}` : hour;
        options.push(`${formattedHour}:00`);
      }
      return options.reverse();
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
    showServiceModal() {
      this.isServiceModalVisible = true;
    },
    loadInitialData() {
      axios.get(`${this.base_url}/api/v1/orders/${this.userId}/`, { headers: { 'ngrok-skip-browser-warning': "oke" } })
        .then(response => {
          const data = response.data;
          if (data.tasks.length === 0) {
            this.editMode = true;
          } else {
            this.services = data.tasks || [];
            this.startTime = data.time_start;
            this.endTime = data.time_end;
            this.workingDays = data.work_days || [];
          }
        })
        .catch(() => {
          this.editMode = true;
        });
    },
    toggleEditMode() {
      this.editMode = !this.editMode;
    },
    resetForm() {
      this.selectedCategory = null;
      this.cost = null;
      console.log("!!!!!");
    },
    removeService(index) {
      this.services.splice(index, 1);
      this.resetForm();
      console.log('After delete service - selectedCategory:', this.selectedCategory, 'cost:', this.cost);
    },
    addService() {
      console.log('Before adding service - selectedCategory:', this.selectedCategory, 'cost:', this.cost);
      if (this.selectedCategory && this.cost) {
        this.services.push({ category: this.selectedCategory, cost: this.cost });
        this.isServiceModalVisible = false;
        this.selectedCategory = null;
        this.cost = null;
      } else {
        alert('Пожалуйста, заполните все поля.');
      }
    },
    onStartTimeChange(value) {
      this.startTime = value;
    },
    onEndTimeChange(value) {
      this.endTime = value;
    },
    onWorkingDaysChange(value) {
      this.workingDays = value;
    },
    onCostChange(value) {
      this.cost = value;
    },
    onCategoryChange(value) {
      this.selectedCategory = value;
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
      try {
        await axios.get(`${this.base_url}/api/v1/orders/${this.userId}/`);
        await axios.put(`${this.base_url}/api/v1/orders/${this.userId}`, dataToSend);
        alert('Информация успешно сохранена');
        this.toggleEditMode(); 
      } catch (error) {
        await axios.post(`${this.base_url}/api/v1/orders/`, dataToSend);
        alert('Информация успешно сохранена');
        this.editMode = 0;
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
h2, h3 {
  color: #333;
}
</style>
