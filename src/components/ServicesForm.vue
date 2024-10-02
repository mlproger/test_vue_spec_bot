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
        <a-button type="primary" @click="showServiceModal" class="mb-4">
          Создать услугу
        </a-button>

        <!-- Модальное окно для добавления услуги -->
        <a-modal
          v-model:visible="isServiceModalVisible"
          title="Создание услуги"
          @ok="addService"
          @cancel="handleServiceModalCancel"
        >
          <a-form layout="vertical">
            <a-form-item label="Категория услуги">
              <a-select v-model="selectedCategory" placeholder="Выберите категорию">
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
          <div style="display: flex; align-items: center;">
            <a-button type="primary" @click="openStartTimePicker">
              {{ startTime ? 'Изменить время' : 'Добавить время' }}
            </a-button>
            <span style="margin-left: 10px;">{{ formattedStartTime }}</span>
          </div>
          <a-modal
            v-model:visible="isStartTimePickerVisible"
            title="Выберите время для начала"
            @cancel="closeTimePicker"
            @ok="closeTimePicker"
          >
            <a-time-picker
              ref="startTimePicker"
              v-model="startTime"
              @change="onStartTimeChange"
              format="HH:mm"
            />
          </a-modal>
        </a-form-item>
        <a-form-item label="Конец">
          <div style="display: flex; align-items: center;">
            <a-button type="primary" @click="openEndTimePicker">
              {{ endTime ? 'Изменить время' : 'Добавить время' }}
            </a-button>
            <span style="margin-left: 10px;">{{ formattedEndTime }}</span>
          </div>
          <a-modal
            v-model:visible="isEndTimePickerVisible"
            title="Выберите время для конца"
            @cancel="closeTimePicker"
            @ok="closeTimePicker"
          >
            <a-time-picker
              ref="endTimePicker"
              v-model="endTime"
              @change="onEndTimeChange"
              format="HH:mm"
            />
          </a-modal>
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
      isServiceModalVisible: false,
      isStartTimePickerVisible: false,
      isEndTimePickerVisible: false,
      days: ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'],
      workingDays: [],
      services: [],
      userId: null,
      editMode: false,
      base_url: "https://73c3-188-243-183-39.ngrok-free.app",
    };
  },
  computed: {
    formattedStartTime() {
      return this.startTime ? this.formatTime(this.startTime) : '';
    },
    formattedEndTime() {
      return this.endTime ? this.formatTime(this.endTime) : '';
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
    openStartTimePicker() {
      this.isStartTimePickerVisible = true;
    },
    openEndTimePicker() {
      this.isEndTimePickerVisible = true;
    },
    closeTimePicker() {
      this.isStartTimePickerVisible = false;
      this.isEndTimePickerVisible = false;
    },
    handleServiceModalCancel() {
      this.isServiceModalVisible = false;
      this.resetForm();
    },
    loadInitialData() {
      axios.get(`${this.base_url}/api/v1/orders/${this.userId}/`, { headers: { 'ngrok-skip-browser-warning': "oke" } })
        .then(response => {
          const data = response.data;
          if (data.tasks.length === 0) {
            this.editMode = true;
          } else {
            this.services = data.tasks || [];
            this.startTime = data.time_start ? this.formatTime(data.time_start) : null;
            this.endTime = data.time_end ? this.formatTime(data.time_end) : null;
            this.workingDays = data.work_days || [];
          }
        })
        .catch(error => {
          this.editMode = true;
          alert(error);
          console.error('Ошибка при загрузке данных:', error);
        });
    },
    formatTime(isoString) {
        if (!isoString) return ''; // Если нет значения, возвращаем пустую строку
        const date = new Date(isoString);
        
        // Проверяем, действительно ли дата корректная
        if (isNaN(date.getTime())) return ''; // Если дата некорректная, возвращаем пустую строку

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
    resetForm() {
      this.selectedCategory = null;
      this.cost = null;
    },
    addService() {
      if (this.selectedCategory && this.cost) {
        this.services.push({ category: this.selectedCategory, cost: this.cost });
        this.resetForm();
        this.isServiceModalVisible = false;
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
        this.toggleEditMode(); 
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
