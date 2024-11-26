<template>
  <v-container>
    <v-row margin="100">
      <v-col>
        <h1>WebSRF_FORM</h1>
      </v-col>
      <v-sheet class="mx-auto" width="300">
        <v-form disabled>
          <v-text-field
            v-model="project.order_number"
            label="SRF No./ № заявки:"
            outlined
          ></v-text-field>
          <v-text-field
            v-model="project.order_date"
            label="Дата заявки:"
            outlined
          ></v-text-field>
        </v-form>
      </v-sheet>
    </v-row>
    <v-row>
      <v-col>
        <v-autocomplete
          v-model="selectedUser"
          :items="userList"
          item-text="full_name"
          item-value="id"
          label="Select or write user"
          return-object
          outlined
          @change="handleUserSelection"
        ></v-autocomplete>

        <div v-if="detailedUserData">
          <p><strong>Selected User Details:</strong></p>
          <p>Full Name: {{ detailedUserData.FIO }}</p>
          <p>IIN: {{ detailedUserData.IIN }}</p>
          <p>Organization Name: {{ detailedUserData.orgName }}</p>
          <p>Organization BIN: {{ detailedUserData.orgBIN }}</p>
          <p>Cost Name: {{ detailedUserData.CostName }}</p>
          <p>Cost Code: {{ detailedUserData.CostCode }}</p>
          <p>Position: {{ detailedUserData.position }}</p>
        </div>
        <div v-else>
          <em>No additional information available for this user.</em>
        </div>
      </v-col>

      <v-col cols="6" class="text-right">
        <v-autocomplete
          v-model="selectedGeneralname"
          :items="uniqueGeneralnames"
          label="Search by Generalname"
          outlined
          @change="resetSelection"
          required
        ></v-autocomplete>

        <v-autocomplete
          v-model="selectedObject"
          :items="filteredObjects"
          item-text="name"
          item-value="uid"
          label="Search Related Objects"
          return-object
          outlined
        ></v-autocomplete>

        <div v-if="selectedObject" class="object-details">
          <h3>Selected Object Details:</h3>
          <p><strong>Name:</strong> {{ selectedObject.name }}</p>
          <p><strong>Code:</strong> {{ selectedObject.code }}</p>
          <p><strong>UID:</strong> {{ selectedObject.uid }}</p>
          <p><strong>Generalname:</strong> {{ selectedObject.Generalname }}</p>
        </div>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-sheet class="mx-auto">
          <v-form>
            <v-textarea
              v-model="project.supplier"
              label="Предлагаемые поставщики"
              outlined
              auto-grow
              clearable
            ></v-textarea>
            <v-textarea
              v-model="project.area"
              label="Участок"
              outlined
              auto-grow
              clearable
            ></v-textarea>
          </v-form>
        </v-sheet>
      </v-col>

      <v-col>
        <v-select
          v-model="project.type_expenditure"
          item-text="name"
          label="Вид статьи расходов"
          return-object
          outlined
          :items="expenditureTypes"
          item-value="id"
          required
          @change="fetchExpenditure"
        ></v-select>

        <div v-if="showClientAutocomplete">
          <v-autocomplete
            v-model="client_code"
            :items="clientList"
            item-text="name"
            item-value="bin"
            label="Клиент"
            return-object
            outlined
            @change="fetchClientDetails"
          ></v-autocomplete>
          <v-textarea
            v-model="project.documents"
            label="Ссылочный документ"
            outlined
            auto-grow
            clearable
          ></v-textarea>
          <div v-if="client_code">
            <p><strong>Selected Client Details:</strong></p>
            <p>Code: {{ client_code.code }}</p>
            <p>Name: {{ client_code.name }}</p>
            <p>BIN: {{ client_code.bin }}</p>
          </div>
        </div>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-text-field
          v-model="project.currency"
          outlined
          label="currency"
          disabled
        ></v-text-field>
      </v-col>
      <v-col>
        <v-select
          v-model="project.extra_project"
          label="Project/Non-project"
          :items="['Project', 'Non-Project']"
          outlined
        ></v-select>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-textarea
          v-model="project.description_works"
          label="Описание работ"
          auto-grow
          outlined
        ></v-textarea>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-data-table
          v-model="selected"
          :headers="headers"
          :items="tableData"
          item-value="id"
          class="elevation-1 rounded-0 no-padding"
          outlined
        >
          <!-- Верхний слот с инструментами -->
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

          <!-- Основное тело таблицы -->
          <template v-slot:body="{ items }">
            <tr v-for="(item, index) in items" :key="item.id">
              <td width="5%">
                <v-text-field
                  v-model="item.id"
                  dense
                  outlined
                  disabled
                  border-e-lg
                  class="rounded-0"
                ></v-text-field>
              </td>

              <td width="20%">
                <v-text-field
                  v-model="item.percentual"
                  dense
                  class="rounded-0"
                  label="Описание"
                  outlined
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

              <!-- <td width="5%">
                <v-select
                  v-model="item.measure"
                  :items="measure"
                  item-text="name"
                  item-value="bin"
                  outlined
                  dense
                  class="rounded-0"
                  label="Ед. изм."
                ></v-select>
              </td> */-->

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
      </v-col>
    </v-row>
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
            <v-data-table
              :headers="headersfor"
              :items="rows"
              class="elevation-1"
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
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
    <v-btn @click="submitForm" color="primary"> Отправить заявку </v-btn>
  </v-container>
</template>

<script>
import axios from "axios";
import data from "../public/data/getcostcentertree.json";
import client from "../public/data/getclientlist.json";
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import service from "../public/data/getservicestree.json";
import measure from "../public/data/getmeasurelist.json";
import moment from "moment";

export default {
  components: { Treeselect },
  data() {
    return {
      project: {
        order_number: "",
        order_date: "",
        supplier: "",
        area: "",
        description_works: "",
        currency: "USD",
        documents: "",
        type_expenditure: "",
        date_shipment: moment(new Date()).format("YYYY-MM-DD"),
      },
      value: null,
      options: service,
      normalizer(node) {
        return {
          id: node.code,
          label: node.name,
          children:
            node.items && node.items.length > 0 ? node.items : undefined,
        };
      },
      selectedItems: [],
      headers: [
        { text: "Item #", value: "id" },
        { text: "Item description", value: "percentual" },
        { text: "Start Date", value: "startDate" },
        { text: "End Date", value: "endDate" },
        { text: "count", value: "count" },
        //{ text: "measure", value: "measure" },-->
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
          // measure: null,
          price: null,
          budgetcode: null,
        },
      ],
      selected: [],
      nextId: 2,
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),

      modal: false,
      menu2: false,

      selectedUser: null,
      userList: [],
      detailedUserData: null, // выдает данные об юзере

      expenditureTypes: [],
      client_code: "", //дает данные клиент кода
      clientList: client,
      measure: measure,
      extra_project: "",
      selectedGeneralname: null,
      selectedObject: null, //выдает данные об выбранном проекте
      projectList: this.flattenData(data),
      loading: false,
      error: null,
      headersfor: [
        { text: "№", value: "id", align: "start" },
        { text: "Описание", value: "description" },
        { text: "Файл", value: "file" },
        { text: "Действия", value: "actions", sortable: false },
      ],
      rows: [], // Данные для таблицы
    };
  },
  computed: {
    uniqueGeneralnames() {
      return [...new Set(this.projectList.map((item) => item.Generalname))];
    },
    filteredObjects() {
      return this.projectList.filter(
        (item) => item.Generalname === this.selectedGeneralname
      );
    },
    showClientAutocomplete() {
      return (
        this.project.type_expenditure &&
        ["Reimbursable_by_Client", "Backcharge"].includes(
          this.project.type_expenditure.name
        )
      );
    },
  },
  mounted() {
    this.fetchUsers();
    this.fetchExpenditure();
  },
  methods: {
    handleSelection(items) {
      console.log("Selected items:", items);
    },
    addRow() {
      this.tableData.push({
        id: this.nextId++,
        percentual: "",
        startDate: "",
        endDate: "",
        count: null,
        measure: null,
        price: null,
        budgetcode: null,
      });
    },
    deleteLastRow() {
      // Удаление последней строки, если она существует
      if (this.tableData.length > 0) {
        this.tableData.pop();
      }
    },
    addRowfile() {
      this.rows.push({
        id: this.rows.length + 1,
        description: "",
        file: null,
      });
    },
    // Удалить строку
    removeRowfile(index) {
      this.rows.splice(index, 1);
    },
    convertFileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result.split(",")[1]);
        reader.onerror = (error) => reject(error);
        reader.readAsDataURL(file);
      });
    },
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
        return results; // Возвращаем массив данных
      } catch (error) {
        console.error("Ошибка при обработке строк:", error);
        throw error; // Пробрасываем ошибку для дальнейшей обработки
      }
    },
    async fetchUsers() {
      try {
        this.loading = true;
        const response = await axios.get("http://intra.isker.kz/api/v1/users");
        this.userList = response.data.map((user) => ({
          ...user,
          full_name: `${user.last_name} ${user.first_name}`.trim(),
        }));
      } catch (error) {
        console.error("Error fetching users:", error);
        this.error = "Failed to load user data.";
      } finally {
        this.loading = false;
      }
    },
    async fetchExpenditure() {
      try {
        const response = await axios.get(
          "http://192.168.0.67/api/v1/websrf/getexpenditure"
        );
        this.expenditureTypes = response.data;
      } catch (error) {
        console.error("Error fetching expenditures:", error);
      }
    },
    async handleUserSelection(user) {
      try {
        if (user) {
          const response = await axios.get(
            `http://192.168.0.67/api/v1/1c/websrf/getuser?iin=${user.username}`
          );
          this.detailedUserData = response.data.data || null;
        }
      } catch (error) {
        console.error("Error fetching detailed user data:", error);
      }
    },
    resetSelection() {
      this.selectedObject = null;
    },
    flattenData(data) {
      return data.reduce((acc, item) => {
        acc.push(item);
        if (item.items && item.items.length) {
          acc.push(...this.flattenData(item.items));
        }
        return acc;
      }, []);
    },
    prepareProject() {
      return {
        supplier: this.project.supplier,
        order_number: this.project.order_number,
        order_date: this.project.order_date,
        description_works: this.project.description_works,
        cost_name: this.selectedObject.name,
        cost_code: this.selectedObject.code,
        //is_draft: this.project.is_draft,
        client_code: this.client_code.name || "",
        extra_project: this.project.extra_project,
        reference_document: this.project.documents,
        area: this.project.area,
        type_expenditure: this.project.type_expenditure.name,
        date_shipment: this.project.date_shipment,
        //work: this.tableData,
      };
    },
    prepareUser() {
      return {
        org_name: this.detailedUserData.orgName,
        autor_name: this.detailedUserData.FIO,
        departament: this.detailedUserData.CostName,
        position: this.detailedUserData.Post,
        contacts: this.selectedUser.phone,
      };
    },

    async submitForm() {
      try {
        const url =
          "http://192.168.0.67/api/v1/websrf/create?iin=" +
          this.detailedUserData.IIN;
        const data123 = {
          project: this.prepareProject(),
          user: this.prepareUser(),
          works: this.tableData,
          docs: await this.handleAllRowsUpload(),
          matrix: { privet: "aa" },
        };
        const response = await axios.post(url, data123, {
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
.no-padding {
  padding: 0 !important;
  margin: 0 !important;
}
</style>
