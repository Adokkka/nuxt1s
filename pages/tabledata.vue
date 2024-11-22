<template>
  <div>
    <button @click="sendBudgetCodes">Отправить данные</button>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      // Пример данных, аналогичный вашему
      tableData: [
        {
          id: 1,
          percentual: null,
          startDate: null,
          endDate: null,
          count: null,
          price: null,
          budgetcode: "00-00023571",
        },
        {
          id: 2,
          percentual: "",
          startDate: "",
          endDate: "",
          count: null,
          price: null,
          budgetcode: "00-00023572",
        },
      ],
    };
  },
  methods: {
    sendBudgetCodes() {
      // Извлекаем все бюджеткоды из массива
      const allBudgetCodes = this.tableData.map((item) => item.budgetcode);

      // Отправляем POST-запрос
      axios
        .post("https://example.com/api/budgetcodes", {
          budgetcodes: allBudgetCodes, // Передаем массив бюджеткодов
        })
        .then((response) => {
          console.log("Ответ сервера:", response.data);
        })
        .catch((error) => {
          console.error("Ошибка при отправке:", error);
        });
    },
  },
};
</script>
