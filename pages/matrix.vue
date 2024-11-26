<template>
  <v-container>
    <v-btn color="primary" @click="openDialog">Добавить данные</v-btn>

    <!-- Диалоговое окно -->
    <v-dialog v-model="dialog" max-width="800px">
      <v-card>
        <v-card-title class="d-flex justify-space-between">
          <span>Добавить данные</span>
          <v-btn icon @click="dialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-data-table
            :headers="headers"
            :items="tableData"
            item-value="uid"
            hide-default-footer
            class="elevation-2"
          >
            <template v-slot:body="{ items }">
              <tbody>
                <tr v-for="(item, index) in items" :key="item.uid">
                  <td>{{ item.name }}</td>
                  <td>
                    <v-select
                      v-model="item.selectedOption"
                      :items="item.options"
                      item-value="uid"
                      item-text="name"
                      label="Выберите значение"
                      outlined
                      dense
                      @click:open="fetchOptions(item)"
                    ></v-select>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-data-table>
        </v-card-text>
        <v-card-actions class="d-flex justify-end">
          <v-btn text @click="dialog = false">Отмена</v-btn>
          <v-btn color="primary" @click="saveData">Сохранить</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      dialog: false,
      headers: [
        { text: "Описание", value: "name" },
        { text: "Выбор", value: "selectedOption", sortable: false },
      ],
      tableData: [], // Данные из GET-запроса
    };
  },
  methods: {
    // Открытие диалога
    openDialog() {
      this.dialog = true;
      this.fetchTableData();
    },

    // GET-запрос для получения основной таблицы
    async fetchTableData() {
      try {
        const response = await fetch(
          "http://nls-1c/HttpService/hs/intra/getmatrixlist"
        ); // Замените на реальный URL
        const data = await response.json();

        // Формируем строки таблицы
        this.tableData = data.map((item) => ({
          ...item,
          selectedOption: null,
          options: [], // Здесь будут храниться опции
        }));
      } catch (error) {
        console.error("Ошибка при получении данных:", error);
      }
    },

    // POST-запрос для загрузки опций в v-select
    async fetchOptions(item) {
      const data = {
        type: this.item.namefull,
      };
      try {
        const response = await this.$axios.$post(
          "http://1c-main/testIvan/hs/intra/getvaluematrixlist/",
          data
        );

        const options = await response.json();

        // Заполняем опции для текущего элемента
        this.$set(
          item,
          "options",
          options.map((option) => ({
            name: option.name, // Текст в v-select
            uid: option.uid, // Уникальное значение
          }))
        );
      } catch (error) {
        console.error(`Ошибка при загрузке опций для ${item.namefull}:`, error);
      }
    },

    // Сохранение выбранных данных
    async saveData() {
      const payload = this.tableData.map((item) => ({
        name: item.name,
        selectedOption: item.selectedOption,
      }));

      console.log("Данные для сохранения:", payload);

      try {
        const response = await fetch(
          "http://1c-main/testIvan/hs/intra/savematrixlist",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(payload),
          }
        );

        if (!response.ok) {
          throw new Error("Ошибка при сохранении данных");
        }

        console.log("Данные успешно сохранены");
      } catch (error) {
        console.error("Ошибка при сохранении данных:", error);
      }

      this.dialog = false;
    },
  },
};
</script>
