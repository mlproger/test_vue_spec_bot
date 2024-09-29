<template>
  <div class="services-form">
    <h2>Форма ввода информации об услугах</h2>
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

      <a-button type="primary" @click="addService" style="margin-top: 10px;">
        Добавить услугу
      </a-button>
    </a-form>

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

    <div style="margin-top: 20px;">
      <h3>Рабочее время</h3>
      <a-form layout="vertical">
        <a-form-item label="Начало">
          <a-time-picker v-model="startTime" format="HH:mm" :input-readonly="true" @click="removeFocus" @focus="removeFocus" @change="onStartTimeChange" />
        </a-form-item>
        <a-form-item label="Конец">
          <a-time-picker v-model="endTime" format="HH:mm" :input-readonly="true" @click="removeFocus" @focus="removeFocus" @change="onEndTimeChange" />
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
    </div>
    <a-button type="primary" @click="saveInfo" style="margin-top: 20px;">
      Сохранить информацию
    </a-button>
  </div>
</template>

<script>
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
    };
  },
  methods: {
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
    addService() {
      if (this.selectedCategory && this.cost !== null) {
        this.services.push({
          category: this.selectedCategory,
          cost: this.cost,
        });

        // Сброс формы после добавления услуги
        this.resetForm();
      } else {
        alert('Пожалуйста, выберите категорию и укажите стоимость');
      }
    },
    removeService(index) {
      this.services.splice(index, 1);
    },
    saveInfo() {
      if (!this.startTime || !this.endTime || this.workingDays.length === 0) {
        alert('Пожалуйста, укажите рабочее время и выберите рабочие дни');
        return;
      }

      console.log(`Рабочее время: с ${this.startTime} до ${this.endTime}, Рабочие дни: ${this.workingDays.join(', ')}`);
      console.log('Список услуг:', this.services);
    },
    resetForm() {
      this.selectedCategory = null;
      this.cost = null;
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
