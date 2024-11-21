<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-data-table
          v-model="selected"
          :headers="headers"
          :items="tableData"
          hide-default-footer
          item-value="id"
          class="elevation-1"
          outlined
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-toolbar-title
                >Таблица с редактированием и диапазоном дат</v-toolbar-title
              >
              <v-spacer></v-spacer>
              <v-btn color="primary" @click="addRow">Добавить строку</v-btn>
              <v-btn color="error" @click="deleteLastRow">
                <v-icon left>mdi-delete</v-icon> Удалить строку
              </v-btn>
            </v-toolbar>
          </template>

          <template v-slot:body="{ items }">
            <tr v-for="(item, index) in items" :key="item.id">
              <td width="5%">
                <v-text-field
                  v-model="item.id"
                  dense
                  disabled
                  outlined
                  class="rounded-0"
                ></v-text-field>
              </td>

              <td width="20%">
                <v-text-field
                  v-model="item.percentual"
                  dense
                  outlined
                  class="rounded-0"
                  label="Описание"
                ></v-text-field>
              </td>

              <td width="10%">
                <v-menu
                  v-model="item.startDateMenu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="item.startDate"
                      readonly
                      dense
                      outlined
                      v-bind="attrs"
                      v-on="on"
                      class="rounded-0"
                      label="Начало"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="item.startDate"
                    no-title
                    scrollable
                    @change="item.startDateMenu = false"
                  >
                    <v-btn text color="primary">OK</v-btn>
                  </v-date-picker>
                </v-menu>
              </td>

              <td width="10%">
                <v-menu
                  v-model="item.endDateMenu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="item.endDate"
                      readonly
                      dense
                      outlined
                      v-bind="attrs"
                      v-on="on"
                      class="rounded-0"
                      label="Конец"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="item.endDate"
                    no-title
                    scrollable
                    @change="item.endDateMenu = false"
                  >
                    <v-btn text color="primary">OK</v-btn>
                  </v-date-picker>
                </v-menu>
              </td>

              <td width="5%">
                <v-text-field
                  v-model="item.count"
                  dense
                  outlined
                  class="rounded-0"
                  label="Количество"
                ></v-text-field>
              </td>

              <td width="5%">
                <v-text-field
                  v-model="item.price"
                  dense
                  outlined
                  class="rounded-0"
                  label="Цена"
                ></v-text-field>
              </td>

              <td width="45%">
                <treeselect
                  v-model="item.budgetcode"
                  :options="options"
                  :normalizer="normalizer"
                  class="mb-6 rounded-0"
                  :disable-branch-nodes="true"
                  :show-count="true"
                  :append-to-body="true"
                  label="Категория"
                />
              </td>
            </tr>
          </template>
        </v-data-table>

        <div v-if="selected.length">
          <h3>Выбранный объект:</h3>
          <ul>
            <li><strong>Code:</strong> {{ selected[0].budgetcode.code }} |</li>
            <li><strong>Name:</strong> {{ selected[0].budgetcode.name }} |</li>
          </ul>
        </div>
      </v-col>
    </v-row>

    <v-btn @click="submitForm" color="primary">Отправить заявку</v-btn>
  </v-container>
</template>

<script>
import axios from "axios";
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import service from "../public/data/getservicestree.json";

export default {
  components: { Treeselect },
  data() {
    return {
      options: service,
      normalizer(node) {
        return {
          id: node.code,
          label: node.name,
          children:
            node.items && node.items.length > 0 ? node.items : undefined,
          ...node,
        };
      },
      headers: [
        { text: "Item #", value: "id" },
        { text: "Item description", value: "percentual" },
        { text: "Start Date", value: "startDate" },
        { text: "End Date", value: "endDate" },
        { text: "count", value: "count" },
        { text: "price", value: "price" },
        { text: "budgetcode", value: "budgetcode" },
      ],
      tableData: [
        {
          id: 1,
          percentual: null,
          startDate: null,
          endDate: null,
          count: null,
          price: null,
          budgetcode: null,
        },
      ],
      selected: [],
    };
  },
  methods: {
    addRow() {
      this.tableData.push({
        id: this.tableData.length + 1,
        percentual: "",
        startDate: "",
        endDate: "",
        count: null,
        price: null,
        budgetcode: null,
      });
    },
    deleteLastRow() {
      if (this.tableData.length > 0) {
        this.tableData.pop();
      }
    },
    sendData() {
      // Отправка данных только для выбранных строк
      if (this.selected.length > 0) {
        return {
          name: this.selected[0].budgetcode.name,
          code: this.selected[0].budgetcode.code,
        };
      }
      return {};
    },
    async submitForm() {
      const dataToSend = this.sendData();

      if (Object.keys(dataToSend).length === 0) {
        alert("Пожалуйста, выберите объект!");
        return;
      }

      try {
        const response = await axios.post("http://192.168.0.67/", dataToSend, {
          headers: {
            "Content-Type": "application/json",
          },
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

<style>
.budget {
  padding: 0;
  margin: 0%;
  width: 100px;
  height: 40px;
}
</style>
