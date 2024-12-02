<template>
  <div>
    <h3>Редактировать данные</h3>
    <div>
      <label>
        Номер:
        <input v-model="formData.Nomer" placeholder="Введите номер" />
      </label>
    </div>
    <div>
      <label>
        Дата:
        <input v-model="formData.data" placeholder="Введите дату" />
      </label>
    </div>
    <div>
      <label>
        Статус:
        <select v-model="formData.status">
          <option value="НеСогласован">НеСогласован</option>
          <option value="Согласован">Согласован</option>
        </select>
      </label>
    </div>
    <button @click="updateData">Сохранить изменения</button>

    <!-- Показ сообщений -->
    <div v-if="formData.error" style="color: red">
      Ошибка: {{ formData.msg }}
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      formData: {
        id: "c70ce0e1-5494-11ef-b8cd-000c290d6635", // Идентификатор записи
        Nomer: "SR-1BCMA-24-00014", // Номер записи
        data: "20240807011224", // Дата
        status: "НеСогласован", // Статус
        error: false,
        msg: "", // Сообщение об ошибке
      },
    };
  },
  methods: {
    async updateData() {
      try {
        // Укажите URL для вашего API
        const url = `http://1c-main/testIvan/hs/intra/GetSRFSatus/${this.formData.id}`;

        // Подготовка данных для отправки
        const requestData = {
          Nomer: this.formData.Nomer,
          data: this.formData.data,
          status: this.formData.status, // Пример: "Согласован"
        };

        // Выполнение PATCH-запроса
        const response = await fetch(url, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(requestData),
        });

        // Обработка ответа
        if (!response.ok) {
          throw new Error(`Ошибка сервера: ${response.status}`);
        }

        const updatedData = await response.json();

        // Обновление локальных данных после успешного обновления на сервере
        this.formData = { ...this.formData, ...updatedData };
        console.log("Данные успешно обновлены:", this.formData);
      } catch (error) {
        // Обработка ошибок
        console.error("Ошибка при обновлении данных:", error);
        this.formData.error = true;
        this.formData.msg = error.message;
      }
    },
  },
};
</script>
