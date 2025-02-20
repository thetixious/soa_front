<template>
  <div>
    <div class="button-group">
      <button @click="currentTask = 'byId'">Получение билета по ID</button>
      <button @click="currentTask = 'byCommentStart'">Получение билета по началу комментария</button>
      <button @click="currentTask = 'byPriceCount'">Получение количества билетов с заданной ценой</button>
    </div>

    <div v-if="currentTask === 'byId'" class="form-group">
      <h2>Получение билета по ID</h2>
      <label for="ticketId">Введите ID билета:</label>
      <input
          id="ticketId"
          type="number"
          v-model="ticketId"
          placeholder="Введите ID билета"
          required
      />
      <button @click="fetchTicketById">Найти</button>

      <div v-if="ticket">
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
          <tr key="ticket.id">
            <td>{{ ticket.name }}</td>
            <td>{{ ticket.id }}</td>
            <td>{{ ticket.price }}</td>
            <td>{{ ticket.creationDate }}</td>
            <td>{{ ticket.type }}</td>
            <td>{{ ticket.comment }}</td>
            <td>X: {{ ticket.coordinates.x }}, Y: {{ ticket.coordinates.y }}</td>
            <td>{{ ticket.person.Color_E }}</td>
            <td>{{ ticket.person.Color_H }}</td>
          </tr>
          </tbody>
        </table>
      </div>
      <div v-else-if="errorMessage" class="error">
        <p>{{ errorMessage }}</p>
      </div>
      <div v-else-if="loading">Загрузка...</div>

    </div>

    <div v-if="currentTask === 'byCommentStart'" class="form-group">
      <h2>Получение билетов по началу комментария</h2>
      <label for="commentStartsWith">Введите начало комментария:</label>
      <input
          id="commentStartsWith"
          type="text"
          v-model="commentStartsWith"
          placeholder="Введите подстроку"
          required
      />
      <button @click="fetchTicketsByCommentStart">Найти</button>

      <div v-if="tickets.length">
        <h3>Результаты:</h3>
        <table border="1" cellspacing="0" cellpadding="10" style="width: 100%; border-collapse: collapse;">
          <thead>
          <tr>
            <th>Имя</th>
            <th>Комментарий</th>
            <th>Цена</th>
            <th>Дата создания</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="ticket in tickets" :key="ticket.id">
            <td>{{ ticket.name }}</td>
            <td>{{ ticket.comment }}</td>
            <td>{{ ticket.price }}</td>
            <td>{{ ticket.creationDate }}</td>
          </tr>
          </tbody>
        </table>
      </div>
      <div v-else-if="errorMessage" class="error">
        <p>{{ errorMessage }}</p>
      </div>
      <p v-if="loading">Загрузка...</p>
      <p v-if="!loading && !tickets.length">Нет данных для отображения</p>
    </div>

    <div v-if="currentTask === 'byPriceCount'" class="form-group">
      <h2>Получение количества билетов с заданной ценой</h2>
      <label for="price">Введите цену:</label>
      <input
          id="price"
          type="number"
          v-model="price"
          placeholder="Введите цену билета"
          required
      />
      <button @click="fetchTicketsByPriceCount">Получить количество</button>

      <!-- Отображение результата -->
      <div v-if="ticketCount !== null">
        <h3>Количество билетов с ценой {{ price }}: {{ ticketCount }}</h3>
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
  name: "TicketHandler",
  data() {
    return {
      currentTask: "byCommentStart",
      ticketId: null,
      commentStartsWith: "",
      tickets: [],
      ticket: null,
      price: null,
      ticketCount: null,
      loading: false,
      errorMessage: null,
    };
  },
  methods: {
    async fetchTicketById() {
      if (!this.ticketId) {
        this.errorMessage = "Пожалуйста, введите ID билета.";
        return;
      }

      this.loading = true;
      this.errorMessage = null;
      this.ticket = null;

      try {
        const response = await axios.get(`https://localhost:8000/tickets/${this.ticketId}`);
        this.ticket = response.data;
      } catch (error) {
        this.errorMessage = error.response?.data?.message || "Произошла ошибка";
      } finally {
        this.loading = false;
      }
    },

    async fetchTicketsByCommentStart() {
      this.loading = true;
      this.errorMessage = null;
      this.tickets = [];

      try {
        const response = await axios.get("https://localhost:20000/tickets/start/comment", {
          params: {commentStartsWith: this.commentStartsWith},
        });
        this.tickets = response.data;
      } catch (error) {
        this.errorMessage = error.response?.data?.message || "Произошла ошибка";
      } finally {
        this.loading = false;
      }
    },

    async fetchTicketsByPriceCount() {
      if (this.price === null || this.price === undefined) {
        this.errorMessage = "Пожалуйста, введите цену.";
        return;
      }

      this.loading = true;
      this.errorMessage = null;
      this.ticketCount = null;

      try {
        const response = await axios.get(`http://localhost:20000/tickets/byPrice/${this.price}`);
        this.ticketCount = response.data;
      } catch (error) {
        this.errorMessage = error.response?.data?.message || "Произошла ошибка";
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
  color: #333;
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

.error {
  color: red;
  margin-top: 10px;
  text-align: center;
}

table {
  margin-top: 20px;
}
</style>