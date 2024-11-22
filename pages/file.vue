<template>
  <v-container>
    <v-row>
      <v-col>
        <v-card>
          <v-card-title class="d-flex justify-space-between">
            <span>Загрузка файлов</span>
            <v-btn color="primary" @click="addRow" outlined>
              <v-icon left>mdi-plus</v-icon> Добавить строку
            </v-btn>
          </v-card-title>
          <v-card-text>
            <v-data-table
              :headers="headers"
              :items="rows"
              hide-default-footer
              class="elevation-2"
            >
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
                      />
                    </td>
                    <td>
                      <v-btn color="error" icon @click="removeRow(index)">
                        <v-icon>mdi-delete</v-icon>
                      </v-btn>
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-data-table>
          </v-card-text>
          <v-card-actions class="d-flex justify-space-between">
            <v-btn
              text
              color="error"
              @click="clearTable"
              :disabled="!rows.length"
            >
              Очистить таблицу
            </v-btn>
            <v-btn
              color="primary"
              @click="handleAllRowsUpload"
              :disabled="!rows.length"
            >
              <v-icon left>mdi-cloud-upload</v-icon> Отправить все
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      headersfor: [
        { text: "№", value: "id", align: "start" },
        { text: "Описание", value: "description" },
        { text: "Файл", value: "file" },
        { text: "Действия", value: "actions", sortable: false },
      ],
      rows: [], // Данные для таблицы
    };
  },
  methods: {
    // Добавить новую строку
    addRow() {
      this.rows.push({
        id: this.rows.length + 1,
        description: "",
        file: null,
      });
    },
    // Удалить строку
    removeRow(index) {
      this.rows.splice(index, 1);
    },
    // Преобразование файла в Base64
    convertFileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result.split(",")[1]);
        reader.onerror = (error) => reject(error);
        reader.readAsDataURL(file);
      });
    },
    // Обработка отправки всех строк
    async handleAllRowsUpload() {
      try {
        const results = await Promise.all(
          this.rows.map(async (row, index) => {
            if (!row.file) {
              throw new Error(`Файл в строке ${index + 1} не выбран`);
            }
            const base64File = await this.convertFileToBase64(row.file);
            return {
              description: row.description,
              file: base64File,
            };
          })
        );

        // POST-запрос для всех строк
        const response = await fetch("http://192.168.0.67", {
          method: "POST",
          headersfor: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ rows: results }),
        });

        if (response.ok) {
          alert("Все файлы успешно отправлены!");
        } else {
          alert("Ошибка при отправке файлов.");
        }
      } catch (error) {
        console.error("Ошибка:", error);
        alert(`Произошла ошибка: ${error.message}`);
      }
    },
  },
};
</script>

<style>
/* Добавьте стили по необходимости */
</style>
