<template>
  <v-row>
    <v-col>
      <v-card>
        <v-card-title class="d-flex justify-space-between">
          <span>Загрузка файлов</span>
          <v-btn color="primary" @click="addRowfile" outlined>
            <v-icon left>mdi-plus</v-icon> Добавить строку
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-data-table :headers="headers" :items="rows" class="elevation-1">
            <template v-slot:body="{ items }">
              <tbody>
                <tr v-for="(item, index) in items" :key="index">
                  <td>{{ index + 1 }}</td>
                  <td>
                    <v-text-field
                      v-model="item.description"
                      label="Описание"
                      outlined
                      dense
                    />
                  </td>
                  <td>
                    <v-file-input
                      v-model="item.file"
                      label="Файл"
                      outlined
                      dense
                      @change="validateFile(item, index)"
                    />
                  </td>
                  <td>
                    <v-btn color="error" icon @click="removeRowfile(index)">
                      <v-icon>mdi-delete</v-icon>
                    </v-btn>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-data-table>
        </v-card-text>
        <v-card-actions class="d-flex justify-space-between">
          <v-btn @click="submitForm" color="primary"> Отправить заявку </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      headers: [
        { text: "№", value: "id", align: "start" },
        { text: "Описание", value: "description" },
        { text: "Файл", value: "file" },
        { text: "Действия", value: "actions", sortable: false },
      ],
      rows: [], // Данные для таблицы
    };
  },
  methods: {
    addRowfile() {
      this.rows.push({
        id: this.rows.length + 1,
        description: "",
        file: null,
      });
    },
    removeRowfile(index) {
      this.rows.splice(index, 1);
    },
    validateFile(item, index) {
      if (!item.file) {
        console.warn(`Файл в строке ${index + 1} не выбран`);
      }
    },
    convertFileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result.split(",")[1]);
        reader.onerror = reject;
        reader.readAsDataURL(file);
      });
    },
    async prepareRows() {
      const preparedRows = [];
      for (const [index, row] of this.rows.entries()) {
        if (!row.file) {
          throw new Error(`Файл в строке ${index + 1} не выбран`);
        }
        const base64File = await this.convertFileToBase64(row.file);
        preparedRows.push({
          id: index + 1,
          description: row.description,
          file: base64File,
          file_name: row.file.name,
        });
      }
      return preparedRows;
    },
    async submitForm() {
      try {
        const url = "http://127.0.0.1:8000/expenditures/";
        //const preparedRows = await this.prepareRows();

        const payload = {
          project: { refdocs: await this.prepareRows() },
        };

        const response = await axios.post(url, payload, {
          headers: { "Content-Type": "application/json" },
        });
        console.log("Заявка успешно отправлена:", response.data);
        alert("Заявка успешно отправлена!");
      } catch (error) {
        console.error("Ошибка при отправке заявки:", error);
        alert("Произошла ошибка при отправке.");
      }
    },
  },
};
</script>
