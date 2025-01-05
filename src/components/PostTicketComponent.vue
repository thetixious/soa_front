<template>
  <div id="ticket-creation">
    <h2>{{ isUpdateMode ? 'Обновить билет' : 'Создать новый билет' }}</h2>

    <div class="switch-container">
      <label class="switch">
        <input type="checkbox" v-model="isUpdateMode" />
        <span class="slider"></span>
      </label>
      <span>{{ isUpdateMode ? 'Обновить существующий билет' : 'Создать новый билет' }}</span>
    </div>


    <div v-if="isUpdateMode">
      <label for="ticketId">ID билета:</label>
      <input v-model.number="ticketData.id" id="ticketId" type="number"  min="1" required />
      <p v-if="validationErrors.id" class="error">{{ validationErrors.id }}</p>
    </div>

    <form @submit.prevent="validateAndSubmit">
      <div>
        <label for="name">Название:</label>
        <input v-model="ticketData.name" id="name" type="text" required />
        <p v-if="validationErrors.name" class="error">{{ validationErrors.name }}</p>
      </div>

      <div>
        <label>Координаты:</label>
        <input v-model.number="ticketData.coordinates.x" placeholder="X" type="number" step="0.0000000001" required />
        <input
            v-model.number="ticketData.coordinates.y"
            placeholder="Y"
            type="number"
            :min="156"
            required
        />
        <p v-if="validationErrors.coordinates" class="error">{{ validationErrors.coordinates }}</p>
      </div>

      <div>
        <label for="price">Цена:</label>
        <input v-model.number="ticketData.price" id="price" type="number" min="1" step="0.0000000001" required />
        <p v-if="validationErrors.price" class="error">{{ validationErrors.price }}</p>
      </div>

      <div>
        <label for="comment">Комментарий:</label>
        <input v-model="ticketData.comment" id="comment" type="text" required />
        <p v-if="validationErrors.comment" class="error">{{ validationErrors.comment }}</p>
      </div>

      <div>
        <label for="type">Тип:</label>
        <select v-model="ticketData.type" id="type" required>
          <option value="" disabled>Выберите тип</option>
          <option value="VIP">VIP</option>
          <option value="USUAL">Usual</option>
          <option value="BUDGETARY">Budgetary</option>
          <option value="CHEAP">Cheap</option>
        </select>
        <p v-if="validationErrors.type" class="error">{{ validationErrors.type }}</p>
      </div>

      <fieldset>
        <legend>Данные о человеке</legend>

        <div>
          <label for="birthday">Дата рождения:</label>
          <input
              v-model="ticketData.person.birthday"
              id="birthday"
              type="date"
              required
          />
          <p v-if="validationErrors.birthday" class="error">{{ validationErrors.birthday }}</p>
        </div>

        <div>
          <label for="color_e">Цвет глаз:</label>
          <select v-model="ticketData.person.Color_E" id="color_e" required>
            <option value="RED">Красный</option>
            <option value="BLUE">Синий</option>
            <option value="ORANGE">Оранжевый</option>
            <option value="WHITE">Белый</option>
          </select>
          <p v-if="validationErrors.Color_E" class="error">{{ validationErrors.Color_E }}</p>
        </div>

        <div>
          <label for="color_h">Цвет волос:</label>
          <select v-model="ticketData.person.Color_H" id="color_h" required>
            <option value="RED">Красный</option>
            <option value="YELLOW">Желтый</option>
            <option value="GREEN">Зелёный</option>
            <option value="WHITE">Белый</option>
          </select>
          <p v-if="validationErrors.Color_H" class="error">{{ validationErrors.Color_H }}</p>
        </div>
      </fieldset>

      <button type="submit">{{ isUpdateMode ? 'Обновить билет' : 'Создать билет' }}</button>
    </form>

    <div v-if="responseMessage" class="response">{{ responseMessage }}</div>
    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
  </div>
</template>

<script>
import axios from "axios";
import Decimal from 'decimal.js';

export default {
  name: "TicketCreation",
  data() {
    return {
      isUpdateMode: false,
      ticketData: {
        id: null,
        name: "",
        coordinates: { x: 0, y: null },
        price: null,
        comment: "",
        type: "",
        person: {
          birthday: "",
          Color_E: "",
          Color_H: "",
        },
      },
      validationErrors: {},
      responseMessage: null,
      errorMessage: null,
    };
  },
  methods: {
    validateAndSubmit() {
      this.validationErrors = {};

      // Валидация
      if (this.isUpdateMode && (!this.ticketData.id || this.ticketData.id <= 0)) {
        this.validationErrors.id = "ID должен быть больше 0.";
      }
      if (!this.ticketData.name) {
        this.validationErrors.name = "Название не может быть пустым.";
      }
      if (!this.ticketData.person.birthday) {
        this.validationErrors.birthday = "Дата рождения обязательна для заполнения.";
      }
      if (
          this.ticketData.coordinates.y === null ||
          this.ticketData.coordinates.y < 156
      ) {
        this.validationErrors.coordinates = "Y должен быть не больше 156.";
      }
      if (this.ticketData.price === null || this.ticketData.price <= 0) {
        this.validationErrors.price = "Цена должна быть больше 0.";
      }
      if (!this.ticketData.comment) {
        this.validationErrors.comment = "Комментарий не может быть пустым.";
      }
      if (!this.ticketData.type) {
        this.validationErrors.type = "Выберите тип билета.";
      }
      if (!this.ticketData.person.Color_E) {
        this.validationErrors.Color_E = "Выберите цвет глаз.";
      }
      if (!this.ticketData.person.Color_H) {
        this.validationErrors.Color_H = "Выберите цвет волос.";
      }

      if (Object.keys(this.validationErrors).length > 0) {
        return;
      }

      this.handleSubmit();
    },
    async handleSubmit() {
      try {
        // Преобразуем числа в строки
        // const ticketDataToSend = {
        //   ...this.ticketData,
        //   coordinates: {
        //     x: this.ticketData.coordinates.x.toString(), // Преобразуем x в строку
        //     y: this.ticketData.coordinates.y, // y остаётся числом
        //   },
        //
        // };
        const url = this.isUpdateMode
            ? `https://localhost:8080/tickets/${this.ticketData.id}`
            : "https://localhost:8080/tickets";
        const method = this.isUpdateMode ? "put" : "post";

        const response = await axios({ method, url, data: this.ticketData });
        this.responseMessage = response.data;
        this.errorMessage = null;
      } catch (error) {
        this.errorMessage = error.response?.data?.message || "Произошла ошибка";
        this.responseMessage = null;
      }
    },
  },
};
</script>
<!--<script>-->
<!--import axios from 'axios';-->

<!--export default {-->
<!--  name: 'TicketCreation',-->
<!--  data() {-->
<!--    return {-->
<!--      isUpdateMode: false, // флаг для переключения между режимами создания и обновления-->
<!--      ticketData: {-->
<!--        id: null,-->
<!--        name: '',-->
<!--        coordinates: { x: 0, y: 0 },-->
<!--        creationDate: new Date().toISOString().split('T')[0], // текущая дата в формате yyyy-mm-dd-->
<!--        price: null,-->
<!--        comment: '',-->
<!--        type: 'VIP',-->
<!--        person: {-->
<!--          birthday: new Date().toISOString().split('T')[0],-->
<!--          Color_E: 'RED',-->
<!--          Color_H: 'GREEN',-->
<!--        },-->
<!--      },-->
<!--      responseMessage: null,-->
<!--      errorMessage: null,-->
<!--    };-->
<!--  },-->
<!--  methods: {-->
<!--    async handleSubmit() {-->
<!--      try {-->
<!--        if (this.isUpdateMode) {-->
<!--          // Отправка запроса на обновление-->
<!--          const response = await axios.put(`http://localhost:8080/tickets/${this.ticketData.id}`, this.ticketData);-->
<!--          this.responseMessage = response.data;-->
<!--        } else {-->
<!--          // Отправка запроса на создание-->
<!--          const response = await axios.post('http://localhost:8080/tickets', this.ticketData);-->
<!--          this.responseMessage = response.data;-->
<!--        }-->
<!--        this.errorMessage = null;-->
<!--      } catch (error) {-->
<!--        this.errorMessage = error.response?.data?.message || 'Произошла ошибка';-->
<!--        this.responseMessage = null;-->
<!--      }-->
<!--    },-->
<!--  },-->
<!--};-->
<!--</script>-->


<style>
/* Container for the switch */
/* Centering container for the switch */
.switch-container {
  display: flex;
  align-items: center; /* Центрирование по вертикали */
  justify-content: center; /* Центрирование по горизонтали */
  gap: 15px; /* Расстояние между переключателем и текстом */
  margin-bottom: 20px;
}

/* The switch */
.switch {
  position: relative;
  display: inline-block;
  width: 50px;
  height: 28px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

/* Slider for the switch */
.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  border-radius: 34px;
  transition: 0.4s;
}

.slider:before {
  position: absolute;
  content: '';
  height: 20px;
  width: 20px;
  left: 4px;
  bottom: 4px;
  background-color: white;
  border-radius: 50%;
  transition: 0.4s;
}

input:checked + .slider {
  background-color: #4caf50;
}

input:checked + .slider:before {
  transform: translateX(22px);
}
#ticket-creation {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

form > div {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
}

input, select, button {
  width: 100%;
  padding: 8px;
  font-size: 16px;
}

button {
  background-color: #42b983;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #35495e;
}

.response {
  color: green;
  margin-top: 15px;
}

.error {
  color: red;
  margin-top: 15px;
}

</style>
