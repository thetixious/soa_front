<template>
  <div id="ticket-list">
    <h2>Список билетов</h2>

    <form @submit.prevent="fetchTickets">
      <div class="filters">
        <h3>Сортировка:</h3>
        <div v-for="(sort, index) in queryParams.sort" :key="index" class="sort-item">
          <select v-model="sort.field">
            <option disabled value="">Выберите поле</option>
            <option v-for="option in sortFieldOptions" :key="option" :value="option">{{ option }}</option>
          </select>
          <select v-model="sort.direction">
            <option value="asc">По возрастанию</option>
            <option value="des">По убыванию</option>
          </select>
          <button @click="removeSort(index)">Удалить</button>
        </div>
        <button @click="addSort">Добавить сортировку</button>
      </div>

      <div v-for="(filter, index) in queryParams.filters" :key="index" class="sort-item">
        <label :for="'filter-field-' + index">Поле:</label>
        <select v-model="filter.field" :id="'filter-field-' + index">
          <option disabled value="">Выберите поле</option>
          <option v-for="option in fieldOptions" :key="option" :value="option">{{ option }}</option>
        </select>

        <label :for="'filter-operation-' + index">Операция:</label>
        <select v-model="filter.operation" :id="'filter-operation-' + index">
          <option disabled value="">Выберите операцию</option>
          <option v-for="option in filterOptions" :key="option" :value="option">{{ option }}</option>
        </select>

        <label :for="'filter-value-' + index">Значение:</label>
        <input
            v-model="filter.value"
            :id="'filter-value-' + index"
            type="text"
            placeholder="Введите значение"
        />

        <button type="button" @click="removeFilter(index)">Удалить</button>
      </div>

      <button type="button" @click="addFilter">Добавить фильтр</button>

      <div class="filters">
        <label for="page">Страница:</label>
        <input v-model.number="queryParams.page" id="page" type="number" min="0" />
      </div>

      <button type="submit">Применить</button>
    </form>

    <!-- Список билетов -->
    <div v-if="tickets.length">
      <h3>Результаты:</h3>
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
        <tr v-for="ticket in tickets" :key="ticket.id">
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
    <div v-else>
      <p v-if="loading">Загрузка...</p>
      <p v-if="!loading && !tickets.length">Нет данных для отображения</p>
    </div>

    <div v-if="errorMessage" class="error">
      <p>{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "TicketList",
  data() {
    return {
      tickets: [],
      queryParams: {
        sort: [],
        filters: [{ value: "" }],
        page: 0,
      },
      sortFieldOptions: [
        "id",
        "creationDate",
        "coordinates.x",
        "coordinates.y",
        "price",
        "name",
        "comment",
        "type",
        "person.colorE",
        "person.colorH"
      ],
      filterOptions: ["gt", "eq", "ne", "lt", "gte", "lte"], // Возможные операции
      fieldOptions: [
        "id", "name", "price", "creationDate", "coordinates.x", "coordinates.y", "comment", "type",
        "person.colorE", "person.colorH"
      ],
      sortOptions: [
        "id_asc", "creationDate_asc", "coordinates.x_asc", "coordinates.y_asc", "price_asc",
        "name_asc", "comment_asc", "type_asc", "person.colorE_asc", "person.colorH_asc",
        "id_des", "creationDate_des", "coordinates.x_des", "coordinates.y_des", "price_des",
        "name_des", "comment_des", "type_des", "person.colorE_des", "person.colorH_des",
      ],
      loading: false,
      errorMessage: null,
    };
  },
  methods: {
    addSort() {
      this.queryParams.sort.push({ field: "", direction: "asc" });
    },
    removeSort(index) {
      this.queryParams.sort.splice(index, 1);
    },

    addFilter() {
      this.queryParams.filters.push({ value: "" });
    },
    removeFilter(index) {
      this.queryParams.filters.splice(index, 1);
    },
    async fetchTickets() {
      this.loading = true;
      this.errorMessage = null;

      try {
        const params = this.formatQueryParams();
        const response = await axios.get(`https://localhost:8080/tickets?${params}`);
        this.tickets = response.data;
      } catch (error) {
        this.errorMessage = error.response?.data?.message || "Произошла ошибка";
      } finally {
        this.loading = false;
      }
    },
    formatQueryParams() {
      const params = [];

      this.queryParams.sort.forEach(sort => {
        if (sort.field && sort.direction) {
          params.push(`sort=${sort.field}_${sort.direction}`);
        }
      });

      this.queryParams.filters.forEach(filter => {
        if (filter.field && filter.operation && filter.value) {
          params.push(`filter=${encodeURIComponent(filter.field)}_${filter.operation}=${encodeURIComponent(filter.value)}`);
        }
      });

      if (this.queryParams.page !== null && this.queryParams.page !== undefined) {
        params.push(`page=${this.queryParams.page}`);
      }

      return params.join("&");
    },
  },
};
</script>
<style scoped>
#ticket-list {
  padding: 20px;
}

.filters {
  margin-bottom: 15px;
}

.sort-item {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 10px;
}

.sort-item label {
  width: 100px;
  text-align: right;
}

.sort-item select,
.sort-item input {
  width: 200px;
  padding: 5px;
  box-sizing: border-box;
}

button {
  padding: 8px 15px;
  margin-top: 5px;
  width: 200px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}
#page {
  width: 200px;
}
button:hover {
  background-color: #45a049;
}

.error {
  color: red;
  margin-top: 20px;
}
</style>
