<template>
  <div>
    <div class="button-group">
      <button @click="currentTask = 'deleteById'">Удалить билет по ID</button>
      <button @click="currentTask = 'deleteByPrice'">Удалить билеты с данной ценой</button>
    </div>

    <div v-if="currentTask === 'deleteById'" class="form-group">
      <h2>Удаление билета по ID</h2>
      <label for="deleteTicketId">Введите ID билета:</label>
      <input
          id="deleteTicketId"
          type="number"
          v-model="ticketId"
          placeholder="Введите ID билета"
          required
      />
      <button @click="deleteTicketById">Удалить</button>

      <div v-if="successMessage">
        <p class="success">{{ successMessage }}</p>
      </div>
      <div v-else-if="errorMessage" class="error">
        <p>{{ errorMessage }}</p>
      </div>
      <div v-else-if="loading">Загрузка...</div>
    </div>

    <div v-if="currentTask === 'deleteByPrice'" class="form-group">
      <h2>Удаление билетов с заданной ценой</h2>
      <label for="deletePrice">Введите цену:</label>
      <input
          id="deletePrice"
          type="number"
          v-model="price"
          placeholder="Введите цену билета"
          required
      />
      <button @click="deleteTicketsByPrice">Удалить</button>

      <div v-if="deletedTicket">
        <h3>Удалённый билет:</h3>
        <table border="1" cellspacing="0" cellpadding="10" style="width: 100%; border-collapse: collapse;">
          <thead>
          <tr>
            <th>Имя</th>
            <th>Идентификатор</th>
            <th>Цена</th>
            <th>Дата создания</th>
            <th>Тип</th>
            <th>Комментарий</th>
            <th>Координаты (X, Y)</th>
            <th>Цвет глаз</th>
            <th>Цвет волос</th>
          </tr>
          </thead>
          <tbody>
          <tr  key="ticket.id">
            <td>{{ deletedTicket.name }}</td>
            <td>{{ deletedTicket.id }}</td>
            <td>{{ deletedTicket.price }}</td>
            <td>{{ deletedTicket.creationDate }}</td>
            <td>{{ deletedTicket.type }}</td>
            <td>{{ deletedTicket.comment }}</td>
            <td>X: {{ deletedTicket.coordinates.x }}, Y: {{ deletedTicket.coordinates.y }}</td>
            <td>{{ deletedTicket.person.Color_E }}</td>
            <td>{{ deletedTicket.person.Color_H }}</td>
          </tr>
          </tbody>
        </table>
      </div>
      <div v-if="successMessage && !deletedTicket">
        <p class="success">{{ successMessage }}</p>
      </div>
      <div v-else-if="errorMessage" class="error">
        <p>{{ errorMessage }}</p>
      </div>
      <div v-else-if="loading">Загрузка...</div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "TicketDeletionHandler",
  data() {
    return {
      currentTask: null,
      ticketId: null,
      price: null,
      loading: false,
      successMessage: null,
      errorMessage: null,
      deletedTicket: null,
    };
  },
  methods: {
    async deleteTicketById() {
      if (this.ticketId === null || this.ticketId === undefined) {
        this.errorMessage = "Пожалуйста, введите ID билета.";
        return;
      }

      this.loading = true;
      this.successMessage = null;
      this.errorMessage = null;

      try {
        await axios.delete(`https://localhost:8080/tickets/${this.ticketId}`);
        this.successMessage = `Билет с ID ${this.ticketId} успешно удалён.`;
      } catch (error) {
        this.successMessage = null
        this.errorMessage = error.response?.data?.message || "Произошла ошибка при удалении билета.";
      } finally {
        this.loading = false;
      }
    },

    async deleteTicketsByPrice() {
      if (this.price === null || this.price === undefined) {
        this.errorMessage = "Пожалуйста, введите цену.";
        return;
      }

      this.loading = true;
      this.successMessage = null;
      this.errorMessage = null;


      try {
        const response = await axios.delete(`https://localhost:8080/tickets/byPrice/${this.price}`);
        this.deletedTicket = response.data;
        this.successMessage = `Билет с ценой ${this.price} успешно удалён.`;
      } catch (error) {
        this.successMessage = null;
        this.deletedTicket = null;
        this.errorMessage = error.response?.data?.message || "Произошла ошибка при удалении билетов.";
      } finally {
        this.loading = false;
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

button:active {
  background-color: #2a7d55;
}

button:focus {
  outline: none;
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

.success {
  color: green;
  margin-top: 10px;
  text-align: center;
}

.error {
  color: red;
  margin-top: 10px;
  text-align: center;
}
</style>
