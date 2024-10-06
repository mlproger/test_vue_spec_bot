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
            Начало рабочего дня: {{ formattedStartTime }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Конец рабочего дня: {{ formattedEndTime }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Рабочие дни: {{ workingDays.join(', ') }}
          </div>
        </a-list-item>
        <a-list-item>
          <div style="flex-grow: 1;">
            Тип заказа: {{ orderType }}
          </div>
        </a-list-item>
        <a-list-item>
          <div>
            <template v-if="selectedCity">
              <div class="city-info">
                <span>Город: {{ selectedCity }}</span>
              </div>
            </template>
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
              <a-select v-model="selectedCategory" placeholder="Выберите категорию" @change="onCategoryChange" class="custom-select">
                <a-select-option v-for="category in categories" :key="category" :value="category">
                  {{ category }}
                </a-select-option>
              </a-select>
            </a-form-item>

            <a-form-item label="Стоимость">
              <a-input-number
                v-model="cost"
                placeholder="Введите стоимость"
                class="centered-input"
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
        <a-form-item label="Начало рабочего дня:">
          <a-select v-model="startTime" @change="onStartTimeChange" placeholder="Выберите время" class="custom-select">
            <a-select-option
              v-for="time in timeOptions"
              :key="time"
              :value="time"
            >
              {{ time }}
            </a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item label="Конец рабочего дня:">
          <a-select v-model="endTime" @change="onEndTimeChange" placeholder="Выберите время" class="custom-select">
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
        
        <a-select v-model="orderType" @change="handleChange" placeholder="Тип заказа" style="width: 200px">
          <a-select-option :value="true">С выездом</a-select-option>
          <a-select-option :value="false">Без выезда</a-select-option>
          </a-select>
        <h3 style="margin-top: 20px; font-weight: bold;">Город</h3>
        <a-form-item>
          <template v-if="selectedCity">
            <div class="city-info">
              <span>{{ selectedCity }}</span>
              <a-button type="link" @click="openCitySelector" class="change-city-btn">Изменить</a-button>
            </div>
          </template>
          <template v-else>
            <a-button @click="openCitySelector">Выбрать город</a-button>
          </template>
        </a-form-item>

        <a-modal
          v-model:visible="isModalVisible"
          title="Выберите город"
          @ok="handleOk"
          width="600px"
          ok-text="Сохранить"
          :cancel-button-props="{ style: { display: 'none' } }"
        >
          <a-select
            v-model:value="selectedCity"
            show-search
            filter-option="false"
            placeholder="Введите название города"
            style="width: 100%"
            @search="onSearch"
            @popupScroll="onScroll"
            :loading="loading"
          >
            <a-select-option
              v-for="city in cities"
              :key="city.id"
              :value="city.name"
            >
              {{ city.name }}
            </a-select-option>
          </a-select>
        </a-modal>
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
      categories: ['Аварийные службы', 'Срочный ремонт' , 'Уборка' ,'Ремонт бытовой техники' , 'Компьютерная помощь' ,'Парикмахерские услуги' ,'Эвакуация автомобилей' ,'Шиномонтаж' ,'Юридические консультации' ,'Переводчики' ,'Праздники'],
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
      base_url: "https://cafe-188-243-183-61.ngrok-free.app",
      orderType: null,
      isModalVisible: false,
      cities: [],
      selectedCity: null,
      loading: false,
      page: 1,
      searchTerm: '',
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
    async openCitySelector() {
      this.isModalVisible = true;
      this.loadCities();
    },
    handleOk() {
      this.isModalVisible = false;
      console.log('Selected city:', this.selectedCity);
    },
    handleCancel() {
      this.isModalVisible = false;
    },
    showServiceModal() {
      this.isServiceModalVisible = true;
    },
    handleChange(value) {
      this.orderType = value;
    },
    async loadCities(reset = false) {
      if (reset) {
        this.cities = [];
        this.page = 1;
      }
      this.loading = true;
      try {
        const response = await axios.get(
          'https://raw.githubusercontent.com/pensnarik/russian-cities/master/russian-cities.json'
        );
        const citiesList = response.data;

        const filteredCities = citiesList
          .filter((city) =>
            city.name.toLowerCase().includes(this.searchTerm.toLowerCase())
          )
          .slice((this.page - 1) * 20, this.page * 20);

        this.cities.push(...filteredCities);
        this.page += 1;
      } catch (error) {
        console.error('Error loading cities:', error);
      } finally {
        this.loading = false;
      }
    },
    onSearch(value) {
      this.searchTerm = value;
      this.loadCities(true);
    },
    onScroll(event) {
      const { target } = event;
      if (target.scrollTop + target.offsetHeight >= target.scrollHeight) {
        this.loadCities();
      }
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
            this.orderType = data.out === true ? "С выездом" : data.out === false ? "Без выезда" : "";
          }
        })
        .catch(() => {
          this.editMode = true;
        });
        axios.get(`${this.base_url}/api/v1/users/${this.userId}`, { headers: { 'ngrok-skip-browser-warning': "oke" } })
        .then(response => {
          const data = response.data;
          this.selectedCity = data;
        });
    },
    toggleEditMode() {
      this.editMode = !this.editMode;
    },
    resetForm() {
      this.selectedCategory = null;
      this.cost = null;
    },
    removeService(index) {
      this.services.splice(index, 1);
      this.resetForm();
    },
    addService() {
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
        out: this.orderType,
        user_id: this.userId.toString(),
      };

      try {
        const response = await axios.get(`${this.base_url}/api/v1/orders/${this.userId}/`, { headers: { 'ngrok-skip-browser-warning': "oke" } });
        if (response.status === 200) {
          await axios.put(`${this.base_url}/api/v1/orders/${this.userId}`, dataToSend, { headers: { 'ngrok-skip-browser-warning': "oke" } });
          await axios.put(`${this.base_url}/api/v1/users/${this.userId}`, {"city": this.selectedCity.toString()}, { headers: { 'ngrok-skip-browser-warning': "oke" } });
          alert("Информация обновлена");
          this.toggleEditMode();
        }
      } catch (error) {

        if (error.response && error.response.status !== 200) {
          await axios.post(`${this.base_url}/api/v1/orders/`, dataToSend, { headers: { 'ngrok-skip-browser-warning': "oke" } });
          await axios.put(`${this.base_url}/api/v1/users/${this.userId}`, {"city": this.selectedCity.toString()}, { headers: { 'ngrok-skip-browser-warning': "oke" } });
          alert("Информация сохранена");
          this.toggleEditMode();
        } 
      }
    },
  },
};
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
.services-form {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #d9d9d9;
  border-radius: 8px;
  background-color: #fafafa;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  font-family: 'Roboto', sans-serif; 
}

.centered-input {
  height: 50px !important; 

} 

.centered-input .ant-input-number-input {
  text-align: center;
}
::v-deep .custom-select .ant-select-selector {
  height: 50px !important; 
  display: flex;
  align-items: center; 
}
::v-deep .custom-select .ant-select-selection-item {
  line-height: 50px; 
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
