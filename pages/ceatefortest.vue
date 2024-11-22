<template>
  <div>
    <input v-model="searchCode" placeholder="Введите бюджет-код" />
    <button @click="searchData">Поиск</button>
    <div v-if="resultData">
      <h3>Результат:</h3>
      <pre>{{ resultData }}</pre>
    </div>
    <div v-else-if="searchCode && !resultData">
      <p>Ничего не найдено для кода {{ searchCode }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchCode: "", // Код для поиска
      resultData: null, // Результаты поиска
      budgetTree: {
        code: "00000000004",
        name: "Услуги",
        groupe: true,
        uid: "e6e533a6-483d-11e7-8485-204747f14401",
        items: [
          {
            code: "00-00009970",
            name: "BUDGET_CODE",
            groupe: true,
            uid: "997e1270-82e3-11ea-a370-0050568d7ea6",
            items: [
              {
                code: "00-00023199",
                name: "010_REVENUE (MAIN WORKS)",
                groupe: true,
                uid: "fd1e4647-7980-11ee-b85b-000c290d6635",
                items: [
                  {
                    code: "00-00023571",
                    name: "REVENUE (MAIN WORKS) (010)",
                    groupe: false,
                    uid: "7d5074cc-7a10-11ee-b85b-000c290d6635",
                    itemscol: 0,
                  },
                ],
                itemscol: 1,
              },
              {
                code: "00-00023200",
                name: "011_REVENUE (VARIATIONS)",
                groupe: true,
                uid: "0489c936-7981-11ee-b85b-000c290d6635",
                items: [
                  {
                    code: "00-00023572",
                    name: "REVENUE (VARIATIONS) (011)",
                    groupe: false,
                    uid: "7d5074d0-7a10-11ee-b85b-000c290d6635",
                    itemscol: 0,
                  },
                ],
                itemscol: 1,
              },
            ],
          },
        ],
      },
    };
  },
  methods: {
    // Рекурсивный поиск
    findByCode(data, code) {
      if (data.code === code) {
        return data;
      }

      if (data.items && Array.isArray(data.items)) {
        for (let item of data.items) {
          const found = this.findByCode(item, code);
          if (found) {
            return found;
          }
        }
      }

      return null;
    },

    // Поиск по введенному коду
    searchData() {
      if (!this.searchCode) {
        alert("Введите бюджет-код!");
        return;
      }

      this.resultData = this.findByCode(this.budgetTree, this.searchCode);
    },
  },
};
</script>
