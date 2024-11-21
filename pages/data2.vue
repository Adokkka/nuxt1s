<template>
  <div>
    <!-- Компонент Treeselect с выбором только одного объекта -->
    <treeselect
      :multiple="false"
      :options="options"
      v-model="selectedValue"
      :disable-branch-nodes="true"
      :normalizer="normalizer"
      search-nested
      :show-count="true"
    />

    <!-- Вывод информации о выбранном объекте -->
    <div v-if="selectedObject" class="selected-data">
      <h3>Выбранный объект:</h3>
      <ul>
        <li>
          <strong>Code:</strong> {{ selectedObject.code }} |
          <strong>Name:</strong> {{ selectedObject.name }} |
          <strong>Group:</strong> {{ selectedObject.groupe ? "Да" : "Нет" }} |
          <strong>UID:</strong> {{ selectedObject.uid }} |
          <strong>ItemsCol:</strong>
          {{ selectedObject.itemscol || "Не указано" }}
        </li>
      </ul>
    </div>
    <p v-else>Нет выбранного объекта.</p>

    <!-- Красная кнопка отправки -->
    <button class="send-button" @click="sendData" :disabled="!selectedObject">
      Отправить данные
    </button>
  </div>
</template>

<script>
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import service from "../public/data/getservicestree.json";

export default {
  components: { Treeselect },
  data: () => ({
    selectedValue: null, // Для хранения выбранного значения (UID)
    options: service, // Данные для Treeselect
    normalizer(node) {
      return {
        id: node.uid, // Используется как уникальный идентификатор
        label: node.name, // Отображаемое имя
        children: node.items && node.items.length > 0 ? node.items : undefined,
        ...node, // Сохраняем остальные свойства для последующего использования
      };
    },
  }),
  computed: {
    selectedObject() {
      // Найдем объект по выбранному UID
      return this.findNodeById(this.options, this.selectedValue);
    },
  },
  methods: {
    // Рекурсивный поиск объекта в древовидной структуре
    findNodeById(nodes, id) {
      for (const node of nodes) {
        if (node.uid === id) {
          return node;
        }
        if (node.items && node.items.length > 0) {
          const found = this.findNodeById(node.items, id);
          if (found) {
            return found;
          }
        }
      }
      return null;
    },

    // Метод отправки данных
    async sendData() {
      const dataToSend = {
        name: this.selectedObject.name,
        code: this.selectedObject.code,
      };

      try {
        const response = await fetch("http://192.168.0.67/api/submit", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(dataToSend),
        });

        if (!response.ok) {
          throw new Error(`Ошибка: ${response.statusText}`);
        }

        const result = await response.json();
        alert("Данные успешно отправлены!");
        console.log("Ответ сервера:", result);
      } catch (error) {
        console.error("Ошибка при отправке:", error);
        alert("Не удалось отправить данные. Проверьте соединение с сервером.");
      }
    },
  },
};
</script>

<style>
/* Стили для красной кнопки */
.send-button {
  background-color: #ff4d4f;
  color: white;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  font-size: 16px;
  margin-top: 20px;
  border-radius: 4px;
}

.send-button:disabled {
  background-color: #ffcccc;
  cursor: not-allowed;
}
</style>
