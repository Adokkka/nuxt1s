<template>
  <v-container>
    <v-row class="mt-4">
      <v-col cols="12" lg="12" md="12" sm="12" mt-4> </v-col>
      <v-col cols="6" lg="6" md="6" sm="6" style="text-align: left">
        <h1>Форма WebSRF</h1>
        <v-hidden v-model="project.is_draft" value="false"></v-hidden>
        <v-text-field
          v-model="project.order_number"
          label="SRF No./ № заявки:"
        ></v-text-field>
      </v-col>
      <v-col cols="6" lg="6" md="6" sm="6" style="text-align: right">
        <v-btn
          class="primary"
          style="
            height: 40px !important;
            font-size: 10px;
            border-radius: 5px;
            margin: 5px;
          "
          @click="goBack"
        >
          <v-icon left dark> arrow_back_ios </v-icon>
          Назад
        </v-btn>

        <v-text-field
          v-model="project.order_date"
          label="Дата заявки:"
        ></v-text-field>
      </v-col>
    </v-row>
    <!-- I part -->
    <hr class="mt-4" />
    <v-row justify="center" class="mt-6">
      <v-col cols="6" lg="6">
        <v-text-field
          v-model="userData.org_name"
          label="Организация"
          readonly
        ></v-text-field>
        <v-text-field v-model="userData.fio" label="ФИО"></v-text-field>
        <v-text-field v-model="userData.cost_name" label="Отдел"></v-text-field>
        <v-text-field v-model="userData.position" label="Должность">
        </v-text-field>
        <v-text-field
          v-model="userData.contact"
          label="Контактный данные"
        ></v-text-field>
        <v-text-field
          v-model="project.supplier"
          label="Предлагаемые поставщики"
        ></v-text-field>
        <v-textarea
          v-model="project.area"
          label="Участок"
          auto-grow
        ></v-textarea>
      </v-col>
      <!-- end I part -->
      <!-- II part -->
      <v-col cols="6" lg="6">
        <v-select
          v-model="project.cost_name"
          label="Номер проекта"
          :items="CostNameList"
          item-text="Generalname"
          item-value="Generalcode"
          required
        ></v-select>
        <v-select
          v-model="project.cost_code"
          label="Наименование проекта"
          :items="CostCodeList"
          item-text="name"
          item-value="code"
          required
        ></v-select>
        <v-select
          v-model="project.type_expenditure"
          label="Вид статьи расходов"
          :items="expenditureTypes"
          item-text="name"
          item-value="id"
          required
        ></v-select>
        <v-select
          v-model="project.client_code"
          label="Клиент"
          :items="clientList"
          item-text="name"
          item-value="bin"
          required
        ></v-select>
        <v-textarea
          v-model="project.reference_document"
          label="Ссылочный документ"
          auto-grow
        ></v-textarea>
        <v-select
          v-model="project.extra_project"
          label="Project/Non-project"
          :items="['Project', 'Non-Project']"
        ></v-select>
        <v-text-field
          v-model="project.date_shipment"
          label="Дата отгрузки"
          type="date"
        >
        </v-text-field>
      </v-col>
    </v-row>
    <v-row justify="center" class="mt-4">
      <v-textarea
        v-model="project.description_works"
        label="Описание работ"
        auto-grow
      ></v-textarea>
    </v-row>
    <v-row justify="center" class="mt-4">
      <v-col cols="12">
        <h3>Ведомость объема работ</h3>
        <v-row v-for="(item, index) in project.works" :key="index">
          <v-col cols="1">
            <v-text-field
              v-model="item.item_number"
              label="№ статьи"
            ></v-text-field>
          </v-col>
          <v-col cols="3">
            <v-text-field
              v-model="item.item_description"
              label="Описание статьи"
            ></v-text-field>
          </v-col>
          <v-col cols="2">
            <v-text-field
              v-model="item.start_date"
              label="Дата начала"
              type="date"
            ></v-text-field>
          </v-col>
          <v-col cols="2">
            <v-text-field
              v-model="item.end_date"
              label="Дата окончания"
              type="date"
            ></v-text-field>
          </v-col>
          <v-col cols="1">
            <v-text-field
              v-model="item.quantity"
              label="Количество"
            ></v-text-field>
          </v-col>

          <v-col cols="1">
            <v-autocomplete
              v-model="item.measure_name"
              :items="measureList"
              item-text="name"
              item-value="name"
              label="Ед. Изм"
              required
            ></v-autocomplete>
          </v-col>
          <v-col cols="2">
            <v-text-field
              v-model="item.budget_cost"
              label="Стоимость заложенная в бюджете (валюта)"
            ></v-text-field>
          </v-col>
          <v-col cols="3" class="mb-5">
            <v-btn color="primary" @click="openDialogService(item)"
              >Услуги</v-btn
            >
            <v-dialog v-model="dialog_service" hide-overlay width="800">
              <v-card>
                <v-card-title>{{ item.name }} - Services</v-card-title>
                <v-card-text>
                  <v-list>
                    <v-list-item
                      v-for="(service, index) in item.nomenclature"
                      :key="index"
                    >
                      {{ index }}
                      {{ service.name }}
                    </v-list-item>
                  </v-list>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" @click="work.dialog = false"
                    >Close</v-btn
                  >
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-col>
        </v-row>
        <v-btn color="primary" @click="addItem">Добавить статью</v-btn>
        <v-btn color="red" @click="removeItem">Удалить статью</v-btn>
      </v-col>
    </v-row>

    <v-row justify="center" class="mt-4 mb-5" no-gutters>
      <v-col cols="12">
        <h3>Ссылочные документы и приложения</h3>
      </v-col>
      <v-container>
        <v-row v-for="(item, index) in project.refdocs" :key="index">
          <v-col cols="1" class="mb-5 mt-5">
            <v-text-field v-model="item.item_number" label="№"></v-text-field>
          </v-col>
          <v-col cols="6" class="mb-5 mt-5">
            <v-text-field
              v-model="item.item_description"
              label="Наеменование"
            ></v-text-field>
          </v-col>
          <v-col cols="5" class="mb-5 mt-5">
            <v-file-input
              v-model="item.file"
              label="Прикрепить файл"
              show-size
              accept="image/*, application/pdf"
            ></v-file-input>

            <a @click="convertAndDownloadPDF(item.file_data)">{{
              item.file_name
            }}</a>
          </v-col>
        </v-row>
      </v-container>

      <v-col cols="12">
        <v-btn color="primary" @click="addRefItem">Добавить поле</v-btn>
        <v-btn color="red" @click="removeRefItem">Удалить поле</v-btn>
      </v-col>
    </v-row>

    <!-- END OF REF DOCS -->
    <v-row justify="right" class="mt-4">
      <!-- END OF CONTENTS group of buttons -->
      <v-group>
        <v-btn color="primary" @click="openWindowMatrix"
          >Матрица ответственности</v-btn
        >
        <v-btn color="primary" @click="submitDraftForm(false)"
          >Сохранить в черновик</v-btn
        >
        <v-btn color="danger" @click="submitDraftForm(true)"
          >Отправить на согласование</v-btn
        >
      </v-group>
    </v-row>
    <!-- Dialog for success -->
    <v-dialog v-model="dialog_success" hide-overlay persistent width="400">
      <v-card color="success" dark>
        <v-card-text
          style="text-align: center; padding: 24px 20px; color: white"
        >
          <v-card-title class="headline">
            Отправка данных на согласование
            <v-spacer></v-spacer>
            <v-btn icon @click="closeDialog">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-card-title>
          <v-card-text>
            Вы действительно хотите отправить данные на согласование?
          </v-card-text>
          <v-btn
            width="100%"
            depressed
            color="success"
            @click="sendFormToApproval"
            >Отправить
          </v-btn>
          <v-btn width="100%" depressed color="error" @click="cancelDialog"
            >Отмена
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
    <!-- Dialog for service list -->

    <!--matrix-responsibility-->
    <v-dialog v-model="dialog_matrix" hide-overlay persistent width="800">
      <v-card>
        <v-card-title class="headline">
          Выбор матрицы ответственности
          <v-spacer></v-spacer>
          <v-btn icon @click="dialog_matrix = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-data-table
            :headers="headers"
            :items="matrixResponse"
            class="elevation-1"
          >
            <template v-slot:body="{ items }">
              <tbody>
                <tr v-for="(item, index) in items" :key="index">
                  <td>{{ item.parent_name }}</td>
                  <td style="width: 300px">
                    <v-select
                      :items="item.data"
                      item-value="uid"
                      item-text="name"
                      label="Select option"
                      v-model="item.selected"
                      style="width: 100% !important"
                    ></v-select>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-data-table>
        </v-card-text>
        <v-card-actions style="text-align: right">
          <v-btn color="success" @click="dialog_matrix = false">ОК </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="dialog_success" hide-overlay persistent width="600">
      <v-card>
        <v-card-title class="headline">
          Отправка данных на сохранение в черновик
          <v-spacer></v-spacer>
          <v-btn icon @click="dialog_success = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          Ваши данные были успешно сохранены в черновик
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="success" @click="dialog_success = false">OK</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- Dialog about field was not filled -->
    <v-dialog v-model="dialog_error" hide-overlay persistent width="400">
      <v-card color="error" dark>
        <v-card-text
          style="text-align: center; padding: 24px 20px; color: white"
        >
          <v-card-title class="headline">
            Ошибка
            <v-spacer></v-spacer>
            <v-btn icon @click="dialog_error = false">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-card-title>
          <!-- todos add what exactly field was not filled -->
          <v-card-text> Пожалуйста, заполните все поля</v-card-text>
          <v-btn
            width="100%"
            depressed
            color="error"
            @click="dialog_error = false"
            >OK
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
    <!-- Dialog for server error -->
    <v-dialog v-model="dialog_server" hide-overlay persistent width="400">
      <v-card>
        <v-card-text style="text-align: center; padding: 24px 20px">
          <v-card-title class="headline">
            Ошибка
            <v-spacer></v-spacer>
            <v-btn icon @click="dialog_server = false">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-card-title>
          <!-- todos add what exactly happend -->
          <v-card-text> Произошла ошибка на сервере</v-card-text>
          <v-btn
            width="100%"
            depressed
            color="error"
            @click="dialog_server = false"
            >OK
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import moment from "moment";
import projects from "~/pages/Projects.vue";
import error from "../layouts/error.vue";

export default {
  data() {
    return {
      user: {
        fio: "",
        iin: "",
        cost_name: "",
        cost_code: "",
        position: "",
        list: "",
      },
      project: {
        id: null,
        supplier: "",
        order_number: "",
        order_date: "",
        description_works: "",
        cost_name: "", //
        cost_code: "", //
        is_draft: false,
        client_code: null,
        extra_project: "",
        reference_document: "",
        area: "",
        type_expenditure: "",
        works: [
          {
            id: 0,
            item_number: "",
            item_description: "",
            start_date: "",
            end_date: "",
            quantity: "",
            budget_cost: "",
            nomenclature: [
              {
                uid: "",
                name: "text",
              },
            ],
            measure_name: "",
            service: [],
          },
        ],
        date_shipment: moment(new Date()).format("YYYY-MM-DD"),
        user: {},
        refdocs: [
          {
            item_number: "",
            item_description: "",
            file: "",
          },
        ],
      },
      expenditureTypes: [],
      CostCodeList: [],
      clientList: [],
      CostNameList: [],
      userData: {},
      dialog_success: false,
      dialog_sent_success: false,
      serviceList: [],
      selected: null,
      dialog_draft_success: false,
      dialog_close: false,
      dialog_service: false,
      dialog_matrix: false,
      dialog_error: false,
      dialog_server: false,

      headers: [
        { text: "DESCRIPTION / НАИМЕНОВАНИЕ", value: "description" },
        { text: "CUSTOMER / ЗАКАЗЧИК", value: "customer" },
      ],
      items: [],
      options: [],
      matrixResponse: [],
      serviceSelected: [],
      measureList: [],
      removeWorksIds: [],
      removeRefdocs: [],
    };
  },
  head() {
    return {
      title: "WebSRF",
    };
  },
  computed: {
    error() {
      return error;
    },
    user() {
      return this.$auth.user;
    },
    matrixSelected() {
      return this.matrixResponse
        .filter((item) => item.selected)
        .map((item) => ({ uid: item.uid, selected: item.selected }));
    },
    prepareNomenclatureCompute() {
      return this.project.works
        .filter((item) => item.nomenclature.length > 0)
        .map((item) => {
          return {
            uid: item.nomenclature.map((nom) => nom.uid),
            name: item.nomenclature.map((nom) => nom.name),
          };
        });
    },
  },
  created() {
    this.init();
    this.getExpenditureTypes();
    this.getCostList();
    this.getClient();
    this.getUser();
    this.getServiceList();
    this.getMeasureList();
    this.getMatrixValuesList();

    // this.setCostName();
  },
  methods: {
    convertAndDownloadPDF(data) {
      if (typeof data !== "string") return;
      const binaryData = atob(data);
      // Создайте массив байт из бинарных данных
      // const byteNumbers = binaryData.length;
      const byteNumbers = new Array(binaryData.length);
      console.log("byteNumbers", byteNumbers);
      for (let i = 0; i < binaryData.length; i++) {
        byteNumbers[i] = binaryData.charCodeAt(i);
      }
      console.log("byteNumbers", byteNumbers);
      const byteArray = new Uint8Array(byteNumbers);
      console.log("byteArray", byteArray);
      // Создайте Blob из бинарных данных
      const blob = new Blob([byteArray], { type: "application/pdf" });

      // Создайте ссылку для скачивания файла
      const url = window.URL.createObjectURL(blob);

      // Создайте ссылку для скачивания и нажмите на нее
      const link = document.createElement("a");
      link.href = url;
      link.download = "Document"; // Укажите имя файла, которое вы хотите использовать
      link.click();

      // Освободите ресурсы
      window.URL.revokeObjectURL(url);
    },
    async getProject() {
      if (typeof this.$route.query.id === "undefined") {
        return;
      }
      const response = await this.$axios.$get(
        "api/v1/websrf/get?id=" + this.$route.query.id
      );
      console.log("Project:", response);
      this.project = response;
    },
    async init() {
      try {
        this.project.id = this.$route.query.id;
        await this.getProject();
        await this.getWorks();
      } catch (error) {
        console.error("Error:", error);
      }
    },
    async getWorks(ids) {
      try {
        // ids = [work_id1, work_id2]
        // request to backend
        if (!ids) {
          return;
        }
        // respones to works[]->project
        const response = await this.$axios.$get(
          "api/v1/websrf/works/get?project_id=" + this.project.id
        );
        this.project.works = response.map((item) => {
          return { ...item };
        });
      } catch (error) {
        console.log("Works error:", error);
      }
    },
    setCostName() {
      this.project.cost_name = this.CostNameList.find(
        (item) => item.Generalcode === this.project.cost_name
      );
      this.project.cost_code = this.CostCodeList.find(
        (item) => item.code === this.project.cost_code
      );
    },
    prepareClient(bin) {
      return this.clientList.find((client) => client.bin === bin);
    },
    prepareMeasure(name) {
      console.log("mesueare name", name);
      if (!name) {
        return null;
      }
      return this.measureList.find((measure) => measure.name === name);
    },
    async getUser() {
      try {
        let auth_user = this.$auth.user;
        console.log("User Auth:", this.$auth.user);
        const response = await this.$axios.$get("api/v1/1c/websrf/getuser", {
          params: {
            iin: auth_user.id_num,
          },
        });

        console.log("User Data:", response);
        this.userData = {
          iin: response.id_num,
          org_name: response.full_name ? response.full_name : "",
          org_bin: response.bin ? response.bin : "",
          fio: response.first_name + " " + response.last_name,
          cost_name: response.cost_name,
          position: response.position,
          contact: response.phone,
        };

        console.log("local:", process.env.local);
        console.log("User Data:", this.userData);
      } catch (error) {
        console.error("Error:", error);
      }
    },
    async getBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = (error) => reject(error);
      });
    },

    async prepareData(send_1c = false) {
      // prepare data
      this.project.client = this.prepareClient(this.project.client);
      this.project.is_draft = send_1c;
      this.project.user = this.userData;
      return {
        id: this.$route.query.id,
        project: this.project,
      };
    },
    prepareProject() {
      return {
        id: this.$route.query.id,
        supplier: this.project.supplier,
        order_number: this.project.order_number,
        order_date: this.project.order_date,
        description_works: this.project.description_works,
        cost_name: this.project.cost_name,
        cost_code: this.project.cost_code,
        is_draft: this.project.is_draft,
        client_code: this.project.client_code,
        extra_project: this.project.extra_project,
        reference_document: this.project.reference_document,
        area: this.project.area,
        type_expenditure: this.project.type_expenditure,
        date_shipment: this.project.date_shipment,
      };
    },
    prepareNomenclature() {
      console.log("prepare Nomenclature:", this.project.works.nomenclature);
      if (typeof this.project.works.nomenclature === "undefined") {
        return [];
      }
      const nomenclature = this.project.works.nomenclature.map((item) => {
        return {
          uid: item.uid,
          name: item.name,
        };
      });
      console.log("Nomenclature:", nomenclature);
      return nomenclature;
    },
    prepareWorks() {
      //this.prepareNomenclature()
      // if all fields are empty string "", return empty array
      if (
        this.project.works.every(
          (item) =>
            item.item_number === "" &&
            item.item_description === "" &&
            item.start_date === "" &&
            item.end_date === "" &&
            item.quantity === "" &&
            item.budget_cost === ""
        )
      ) {
        return [];
      }
      const works = this.project.works.map((item) => {
        return {
          id: item.id ? item.id : null,
          item_number: item.item_number,
          item_description: item.item_description,
          start_date: item.start_date,
          end_date: item.end_date,
          quantity: item.quantity,
          budget_cost: item.budget_cost,
          nomenclature: this.prepareNomenclature(),
          measure_name: item.measure_name,
          measure_code: this.prepareMeasure(item.measure_name).code,
        };
      });
      return {
        project_id: this.$route.query.id ? this.$route.query.id : null,
        items: works,
      };
    },
    async prepareRefdocs() {
      // Prepare refdocs with Base64-encoded files
      // if all fields are empty string "", return empty array
      if (
        this.project.refdocs.every(
          (doc) =>
            doc.item_number === "" &&
            doc.item_description === "" &&
            doc.file === ""
        )
      ) {
        return [];
      }
      const refDocsWithFiles = await Promise.all(
        this.project.refdocs.map(async (doc) => {
          const fileBase64 = doc.file ? await this.getBase64(doc.file) : null;
          return {
            item_number: doc.item_number,
            item_description: doc.item_description,
            file: fileBase64 ? { name: doc.file.name, file: fileBase64 } : null,
            file_name: doc.file_name ? doc.file_name : null,
          };
        })
      );
      console.log("Refdocs:", refDocsWithFiles);
      this.project.refdocs = this.project.refdocs.map((doc, index) => {
        const items = { ...doc };
        if (refDocsWithFiles[index].file) {
          items.file_data = refDocsWithFiles[index].file.file;
          items.file_name = refDocsWithFiles[index].file.name;
        }
        return items;
      });
      return this.project.refdocs;
    },
    async submitDraftForm(send_1c) {
      const is_update = this.project.id ? true : false;
      // Prepare data
      // update project
      const data = await this.prepareData(send_1c);
      if (is_update) {
        await this.updateForm(data);
      } else {
        await this.createForm(data);
      }
      // create project
    },
    async putData(url, data) {
      try {
        console.log("put data URL:", url);
        const response = await this.$axios.$put(url, data, {
          headers: {
            "Content-Type": "application/json",
          },
        });
        if (response.ok) {
          this.dialog_draft_success = true;
        } else {
          console.error("Error submitting form, putData");
        }
      } catch (error) {
        console.error("Error put Data:", error);
      }
    },
    async deleteData(url, data) {
      try {
        console.log("put data URL:", url);
        const response = await this.$axios.$delete(url, {
          params: data,
          headers: {
            "Content-Type": "application/json",
          },
        });
        if (response.ok) {
          this.dialog_draft_success = true;
        } else {
          console.error("Error submitting form, putData");
        }
      } catch (error) {
        console.error("Error put Data:", error);
      }
    },
    async updateForm(data) {
      console.log("UPdate Data:", data);
      const project_url = "api/v1/websrf/update?iin=" + this.$auth.user.id_num;
      const works_url =
        "api/v1/websrf/works/update?iin=" + this.$auth.user.id_num;
      const refdocs_url =
        "api/v1/websrf/refdocs/update?iin=" + this.$auth.user.id_num;
      const matrix_url =
        "api/v1/websrf/matrix/update?iin=" + this.$auth.user.id_num;
      const works_delete_url =
        "api/v1/websrf/works/delete?iin=" + this.$auth.user.id_num;
      const refdocs_delete_url =
        "api/v1/websrf/refdocs/delete?iin=" + this.$auth.user.id_num;
      try {
        await this.putData(project_url, this.prepareProject());
        await this.putData(works_url, this.prepareWorks());
        const docs = await this.prepareRefdocs();
        await this.putData(refdocs_url, {
          project_id: this.$route.query.id,
          docs: docs,
        });
        await this.putData(matrix_url, this.prepareMatrix());
        console.log("delete works:", this.removeWorksIds);
        if (this.removeWorksIds.length > 0) {
          await this.deleteData(works_delete_url, {
            project_id: this.$route.query.id,
            ids: this.removeWorksIds,
          });
        }
        if (this.removeRefdocs.length > 0) {
          console.log("delete refdocs:", this.removeRefdocs);
          await this.deleteData(refdocs_delete_url, {
            project_id: this.$route.query.id,
            ids: this.removeRefdocs,
          });
        }
        // await this.putData(refdocs_url, data.project.refdocs)
        // await this.putData(matrix_url, data.matrix)
        this.dialog_draft_success = true;
      } catch (error) {
        console.error("Error:", error);
      }
    },

    async createForm(data) {
      const url = "api/v1/websrf/create?iin=" + this.project.user.iin;
      try {
        const data = {
          project: this.prepareProject(),
          works: this.prepareWorks(),
          refdocs: await this.prepareRefdocs(),
          matrix: this.prepareMatrix(),
          user: this.project.user,
        };
        const response = await this.$axios.$post(url, data, {
          headers: {
            "Content-Type": "application/json",
          },
        });
        if (response.id) {
          console.log("data sent");
          this.dialog_draft_success = true;
          this.project.id = response.id;
          // redirect to /WebsrfList
          this.goBack();
        } else {
          this.dialog_error = true;
          console.error("Error submitting form");
        }
      } catch (error) {
        this.dialog_server = true;
        console.error("Error:", error);
      }
    },
    async submitFormToApproval() {
      this.project.is_draft = true;
      const data = {
        user: this.userData,
        project: this.project,
      };
      try {
        const response = await this.$axios.$post(
          "/api/v1/websrf/sendapproval?iin=" + data.user.iin,
          data
        );
        if (response.ok) {
          this.dialog_success = true;
        } else {
          console.error("Error submitting form");
        }
      } catch (error) {
        console.error("Error:", error);
      }
    },

    //get expenditure types
    async getExpenditureTypes() {
      try {
        const response = await this.$axios.$get(
          "/api/v1/websrf/getexpenditure"
        );
        console.log("Expenditure types:", response);
        if (response) {
          this.expenditureTypes = response.map((item) => {
            return {
              id: item.id,
              name: item.name,
            };
          });
          console.log("Expenditure types:", this.expenditureTypes.name);
          console.log("Expenditure types:", this.expenditureTypes.id);
        } else {
          console.error("Error getting expenditure types");
        }
      } catch (error) {
        console.error("Error:", error);
      }
    },
    async getData(url, params = {}) {
      try {
        const response = await this.$axios.$get(url, {
          params: params,
        });
        return response;
      } catch (error) {
        console.error("Error:", error);
      }
    },
    async getCostList() {
      const url = "/api/v1/1c/websrf/getcostcenter";
      const response = await this.getData(url);

      this.CostCodeList = response.map((item) => {
        return {
          code: item.code,
          name: item.name,
        };
      });
      this.CostNameList = response.map((item) => {
        return {
          Generalcode: item.Generalcode,
          Generalname: item.Generalname,
        };
      });
      console.log("cost code cost:", this.CostCodeList);
      console.log("cost name cost:", this.CostNameList);
    },
    async getClient() {
      const url = "/api/v1/1c/websrf/getclient";
      const response = await this.getData(url);
      this.clientList = response.map((item) => {
        return {
          bin: item.bin,
          name: item.name,
          code: item.code,
        };
      });
      console.log("Client:", response);
    },
    addItem() {
      this.project.works.push({
        item_number: "",
        item_description: "",
        start_date: moment(new Date()).format("YYYY-MM-DD"),
        // add 7 days to current date
        end_date: moment(new Date()).format("YYYY-MM-DD"),
        quantity: "",
        unit: "",
        budget_cost: "",
        nomenclature: [
          {
            uid: "",
            name: Array.from(
              { length: 5 },
              () => "abcde"[Math.floor(Math.random() * 5)]
            ).join(""),
          },
        ],
      });
    },
    removeItem() {
      if (this.project.works.length > 0) {
        this.project.works.pop();
        // collect id to array stack
        this.removeWorksIds.push(
          this.project.works[this.project.works.length - 1].id
        );
      }
    },
    addRefItem() {
      this.project.refdocs.push({
        item_number: "",
        item_description: "",
        file: "",
      });
    },
    removeRefItem() {
      if (this.project.refdocs.length > 0) {
        this.project.refdocs.pop();
        this.removeRefdocs.push(
          this.project.refdocs[this.project.refdocs.length - 1].id
        );
      }
    },
    goBack() {
      // Your goBack logic
      this.$router.push(this.localePath("/WebsrfList"));
    },
    closeDialog() {
      this.dialog_close = false;
    },
    closeMatrixDialog() {
      this.dialog_matrix = false;
    },
    openWindowMatrix() {
      this.dialog_matrix = true;
    },
    openWindowApproval() {
      this.dialog_success = true;
    },
    sendFormToApproval() {
      this.project.is_draft = false;
      this.submitForm();
      this.dialog_success = false;
    },
    cancelDialog() {
      this.dialog_close = false;
    },
    cancelMatrixDialog() {
      this.dialog_matrix = false;
    },
    async getServiceList() {
      const url = "api/v1/websrf/getNomencalture";
      const response = await this.getData(url);
      console.log("Service:", response);
      this.serviceList = response;
    },
    async getMeasureList() {
      const url = "api/v1/websrf/getMeasures";
      const response = await this.getData(url);
      this.measureList = response;
    },
    onUpdateServiceSelected(selected) {
      // Ensure only one item is selected
      if (Array.isArray(selected) && selected.length > 1) {
        // Keep only the last selected item
        this.serviceSelected = [selected[selected.length - 1]];
      } else {
        this.serviceSelected = selected;
      }
      console.log("Selected:", this.serviceSelected, selected);
    },
    onUpdateMeasure(selected) {
      console.log("Selected:", selected);
      if (Array.isArray(selected) && selected.length > 1) {
        this.project.works.measure = [selected[selected.length - 1]];
      } else {
        this.project.works.measure = selected;
      }
    },
    async getMatrixValuesList() {
      const url = "api/v1/websrf/getmatrixvalues";
      const data = await this.getData(url);
      this.matrixResponse = data.map((item) => {
        return {
          parent_name: item.name,
          data: item.data.map((data) => {
            return {
              uid: data.uid,
              name: data.name,
            };
          }),
          namefull: item.namefull,
          selected: null,
        };
      });
    },
    prepareMatrix() {
      // send data by selected uid selectedMatrix [{uid: xxx, name: xxx}] to backend
      // this.matrixSelected => this.matrixResponse => get items that selected
      const selectedItems = this.matrixResponse
        .map((item) => {
          return {
            parent_name: item.parent_name,
            selected: item.data.find((data) => data.uid === item.selected),
            namefull: item.namefull,
          };
        })
        .filter((item) => item.selected !== undefined);
      //selectedItems.filter(item => item.selected !== undefined)
      // update namefull from this.matrixResponse

      // if selected field is not undefined, return selectedItems
      console.log("Selected:", selectedItems);
      return selectedItems;
    },
    openDialogService(index) {
      this.dialog_service = true;
    },
    onUpdateActive(selected) {
      console.log("Selected:", selected);
      if (selected.length > 1) {
        // Deselect the previously selected item
        this.serviceSelected = [selected[selected.length - 1]];
      } else {
        this.serviceSelected = selected;
      }
      this.lastSelected = selected[selected.length - 1];
    },
    onUpdateOpen(open) {
      // Handle open state if needed
    },
  },
};
</script>

<style scoped>
.error {
  color: red;
}
</style>
