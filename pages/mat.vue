<template>
  <div>
    <h1>Список пользователей</h1>

    <!-- Vuetify v-data-table -->
    <v-container>
      <v-data-table
        :headers="headers"
        :items="users"
        item-value="id"
        class="elevation-1"
        dense
      >
        <template v-slot:item.actions="{ item }">
          <v-btn color="primary" @click="selectUser(item)">Выбрать</v-btn>
        </template>
      </v-data-table>

      <!-- v-select для отображения данных после POST-запроса -->
      <div v-if="responseOptions.length" class="mt-5">
        <v-select
          v-model="selectedOption"
          :items="responseOptions"
          item-text="label"
          item-value="value"
          label="Выберите значение"
          outlined
        ></v-select>
        <p class="mt-3">Выбранное значение: {{ selectedOption }}</p>
      </div>
    </v-container>
  </div>
</template>

<script>
import axios from "axios";
import matrix from "../public/data/getmatrixlist.json";
export default {
  data() {
    return {
      headers: [
        { text: "ID", value: "id" },
        { text: "Имя", value: "name" },
        { text: "Действия", value: "actions", sortable: false },
      ],
      users: matrix,
      responseOptions: [], // Данные для v-select после POST-запроса
      selectedOption: null, // Выбранный элемент из v-select
    };
  },
  methods: {
    selectUser(user) {
      // Отправляем POST-запрос с именем пользователя
      axios
        .post(
          "http://1c-main/testIvan/hs/intra/getvaluematrixlist/",
          { type: user.namefull },
          {
            headers: { "Content-Type": "application/json" },
          }
        )
        .then((response) => {
          // Сохраняем данные для v-select (например, response.data.options)
          if (response.data && Array.isArray(response.data.options)) {
            this.responseOptions = response.data.options.map((item) => ({
              label: item.label, // Отображаемый текст
              value: item.value, // Значение элемента
            }));
            alert(`Запрос успешен для пользователя: ${user.name}`);
          } else {
            alert("Ответ от сервера не содержит ожидаемых данных.");
          }
        })
        .catch((error) => {
          console.error("Ошибка запроса:", error);
          alert("Ошибка при выполнении запроса!");
        });
    },
  },
};
</script>

<style scoped>
h1 {
  font-size: 24px;
  margin-bottom: 20px;
}
</style>
