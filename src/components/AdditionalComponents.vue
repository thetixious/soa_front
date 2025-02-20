<template>
  <div id="booking-actions">
    <div class="button-group">
      <button @click="currentTask = 'createDiscount'">Создать билет со скидкой</button>
      <button @click="currentTask = 'cancelBookings'">Отменить бронирования</button>
    </div>

    <div v-if="currentTask === 'createDiscount'" class="form-group">
      <h2>Создание билета со скидкой</h2>
      <form @submit.prevent="createTicketWithDiscount">
        <label for="ticketId">ID билета:</label>
        <input v-model.number="discountForm.ticketId" id="ticketId" type="number" min="1" required />

        <label for="personId">ID человека:</label>
        <input v-model.number="discountForm.personId" id="personId" type="number" min="1" required />

        <label for="discount">Скидка (%):</label>
        <input v-model.number="discountForm.discount" id="discount" type="number" min="0" max="100" step="0.1" required />

        <button type="submit">Создать</button>
      </form>
    </div>

    <div v-if="currentTask === 'cancelBookings'" class="form-group">
      <h2>Отмена бронирований</h2>
      <form @submit.prevent="cancelBookings">
        <label for="personIdCancel">ID человека:</label>
        <input v-model.number="cancelForm.personId" id="personIdCancel" type="number" min="1" required />

        <button type="submit">Отменить</button>
      </form>
    </div>

    <div v-if="responseMessage" class="response">
      <p>{{ responseMessage }}</p>
    </div>
    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "BookingActions",
  data() {
    return {
      currentTask: null,
      discountForm: {
        ticketId: null,
        personId: null,
        discount: null,
      },
      cancelForm: {
        personId: null,
      },
      responseMessage: null,
      errorMessage: null,
    };
  },
  methods: {
    async createTicketWithDiscount() {
      try {
        const { ticketId, personId, discount } = this.discountForm;
        const url = `http://localhost:8081/soa2_1-1.0-SNAPSHOT/api/booking/sell/discount/${ticketId}/${personId}/${discount}`;
        const response = await axios.post(url);
        this.responseMessage = `Билет со скидкой успешно создан для человека с ID ${personId}`;
        this.errorMessage = null;
      } catch (error) {
        this.errorMessage = error.response?.data?.message || "Произошла ошибка при создании билета";
        this.responseMessage = null;
      }
    },
    async cancelBookings() {
      try {
        const { personId } = this.cancelForm;
        const url = `http://localhost:8080/api-1.0-SNAPSHOT/api/booking/person/${personId}/cancel`;
        const response = await axios.put(url);
        // Успешное сообщение при отмене бронирований
        this.responseMessage = `Все бронирования отменены для пользователя с id ${personId}`;
        this.errorMessage = null;
      } catch (error) {
        this.responseMessage = `Все бронирования отменены для пользователя с id ${personId}`;
        this.errorMessage = error.response?.data?.message || "Произошла ошибка при отмене бронирований";
        this.responseMessage = null;
      }
    },
  },
};
</script>

<style scoped>
.button-group {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  font-size: 14px;
  cursor: pointer;
  border: 1px solid #369c70;
  background-color: #42b983;
  color: #fff;
}

button:hover {
  background-color: #369c70;
}

.form-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

input {
  width: 300px;
  padding: 5px;
  margin-bottom: 10px;
}

.response {
  margin-top: 1em;
  color: green;
}

.error {
  margin-top: 1em;
  color: red;
}
</style>
