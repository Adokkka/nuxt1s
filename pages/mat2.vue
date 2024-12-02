<template>
  <div>
    <!-- Кнопка для открытия диалогового окна -->
    <v-btn color="primary" @click="dialog = true">Открыть таблицу</v-btn>

    <!-- Всплывающее окно -->
    <v-dialog v-model="dialog" max-width="800px">
      <v-card>
        <v-card-title>Таблица с выбором</v-card-title>
        <v-card-text>
          <v-simple-table>
            <thead>
              <tr>
                <th>Имя</th>
                <th>Выбор</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, index) in matrixtable" :key="row.uid">
                <td>{{ row.name }}</td>
                <td>
                  <v-select
                    :items="row.items"
                    item-text="name"
                    item-value="uid"
                    v-model="selectedValues[index]"
                    label="Выберите"
                  ></v-select>
                </td>
              </tr>
            </tbody>
          </v-simple-table>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" text @click="dialog = false">Закрыть</v-btn>
          <v-btn color="green" text @click="saveSelection">Сохранить</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Отображение нового массива -->
    <div v-if="matrix.length">
      <h3>Новый массив:</h3>
      <pre>{{ matrix }}</pre>
    </div>
  </div>
</template>

<script>
import matrix from "../public/data/getmatrixlistwithitems.json";
export default {
  data() {
    return {
      dialog: false,
      matrixtable: matrix,
      selectedValues: [], // Хранение выбранных UID для каждой строки
      matrix: [], // Новый массив
    };
  },
  methods: {
    saveSelection() {
      // Генерация нового массива
      this.matrix = this.matrixtable.map((row, index) => {
        const selectedItem = row.items.find(
          (item) => item.uid === this.selectedValues[index]
        );
        return {
          element_name: row.namefull,
          value_name: selectedItem ? selectedItem.name : null,
          value_uid: selectedItem ? selectedItem.uid : null,
        };
      });

      // Закрыть диалог
      this.dialog = false;

      // Лог для проверки
      console.log("Новый массив matrix:", this.matrix);
    },
  },
};
</script>

<style>
/* Дополнительные стили */
</style>
