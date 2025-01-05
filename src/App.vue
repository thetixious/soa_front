<template>
  <div id="app">
    <!-- Главное меню с кнопками -->
    <div class="button-group">
      <button @click="currentComponent = 'GetTicketsComponent'">Получить билеты</button>
      <button @click="currentComponent = 'PostTicketComponent'">Добавить билет</button>
      <button @click="currentComponent = 'GetWithCustomLogic'">Кастомная логика</button>
      <button @click="currentComponent = 'DeleteOperationWithTicketComponent'">Удаление билетов</button>
      <button @click="currentComponent = 'AdditionalComponents'">Дополнительные операции</button>
    </div>

    <!-- Условный рендеринг компонентов -->
    <div class="content">
      <GetTicketsComponent v-if="currentComponent === 'GetTicketsComponent'" />
      <PostTicketComponent v-if="currentComponent === 'PostTicketComponent'" />
      <GetWithCustomLogic v-if="currentComponent === 'GetWithCustomLogic'" />
      <DeleteOperationWithTicketComponent v-if="currentComponent === 'DeleteOperationWithTicketComponent'" />
      <AdditionalComponents v-if="currentComponent === 'AdditionalComponents'" />
    </div>
  </div>
</template>

<script>
import './App.css';
import GetTicketsComponent from "@/components/GetTicketsComponent.vue";
import PostTicketComponent from "@/components/PostTicketComponent.vue";
import GetWithCustomLogic from "@/components/GetWithCustomLogic.vue";
import DeleteOperationWithTicketComponent from "@/components/DeleteOperationWithTicketComponent.vue";
import AdditionalComponents from "@/components/AdditionalComponents.vue";

export default {
  name: 'App',
  components: {
    PostTicketComponent,
    GetTicketsComponent,
    GetWithCustomLogic,
    DeleteOperationWithTicketComponent,
    AdditionalComponents
  },
  data() {
    return {
      currentComponent: null // Управляет отображаемым компонентом
    };
  }
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

button:focus {
  outline: none;
}

.content {
  margin-top: 20px;
  display: flex;
  justify-content: center;
}
</style>
