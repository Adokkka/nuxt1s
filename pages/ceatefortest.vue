<template>
  <v-container>
    <v-row margin="100">
      <v-col>
        <h1>WebSRF_FORM</h1>
      </v-col>
      <v-sheet class="mx-auto" width="300">
        <v-form disabled>
          <v-text-field
            v-model="firstName"
            label="WEBSRF NO."
            outlined
          ></v-text-field>
          <v-text-field
            v-model="date"
            label="WEBSRF Date"
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
              v-model="Proposedsuppliers"
              label="Предлагаемые поставщики"
              outlined
              auto-grow
              clearable
            ></v-textarea>
            <v-textarea
              v-model="Area"
              label="Участок"
              outlined
              auto-grow
              clearable
            ></v-textarea>
          </v-form>
        </v-sheet>
      </v-col>

      <v-col>
        <v-autocomplete
          v-model="type_expenditure"
          item-text="name"
          label="Вид статьи расходов"
          return-object
          outlined
          :items="expenditureTypes"
          item-value="id"
          required
          @change="fetchExpenditure"
        ></v-autocomplete>

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
            v-model="documents"
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
          v-model="reference_document"
          outlined
          label="USD"
          disabled
        ></v-text-field>
      </v-col>
      <v-col>
        <v-select
          v-model="extra_project"
          label="Project/Non-project"
          :items="['Project', 'Non-Project']"
          outlined
        ></v-select>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <v-textarea
          v-model="description_works"
          label="Описание работ"
          auto-grow
          outlined
        ></v-textarea>
      </v-col>
    </v-row>
    <v-row>
      <v-col col="2">
        <v-data-table
          v-model="selected"
          :headers="headers"
          :items="tableData"
          item-value="id"
          class="elevation-1"
          dense
          outlined
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-toolbar-title>Editable Table with Date Range</v-toolbar-title>
              <v-spacer></v-spacer>
              <v-btn color="primary" @click="addRow">Add Row</v-btn>
            </v-toolbar>
          </template>
          <template v-slot:body="{ items }">
            <tr v-for="(item, index) in items" :key="item.id">
              <td>
                <v-text-field v-model="item.id" dense disabled></v-text-field>
              </td>
              <td>
                <v-text-field v-model="item.percentual" dense></v-text-field>
              </td>
              <td>
                <v-text-field v-model="item.label" dense></v-text-field>
              </td>
              <td>
                <v-text-field v-model="item.icon" dense></v-text-field>
              </td>
              <td>
                <v-menu
                  v-model="item.startDateMenu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                  min-width="290px"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="item.startDate"
                      readonly
                      dense
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    outlined
                    v-model="item.startDate"
                    no-title
                    scrollable
                  >
                    <v-spacer></v-spacer>
                    <v-btn
                      outlined
                      text
                      color="primary"
                      @click="item.startDateMenu = false"
                      >OK</v-btn
                    >
                  </v-date-picker>
                </v-menu>
              </td>
              <td>
                <v-menu
                  v-model="item.endDateMenu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  offset-y
                  min-width="290px"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="item.endDate"
                      readonly
                      dense
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker v-model="item.endDate" no-title scrollable>
                    <v-spacer></v-spacer>
                    <v-btn
                      text
                      color="primary"
                      @click="item.endDateMenu = false"
                      >OK</v-btn
                    >
                  </v-date-picker>
                </v-menu>
              </td>
              <td>
                <v-btn icon @click="deleteRow(index)">
                  <v-icon color="red">mdi-delete</v-icon>
                </v-btn>
              </td>
            </tr>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import data from "../public/data/getcostcentertree.json";
import client from "../public/data/getclientlist.json";

export default {
  data() {
    return {
      headers: [
        { text: "ID", value: "id", align: "start" },
        { text: "Value", value: "percentual" },
        { text: "Label", value: "label" },
        { text: "Icon", value: "icon" },
        { text: "Start Date", value: "startDate" },
        { text: "End Date", value: "endDate" },
        { text: "Actions", value: "actions", sortable: false },
      ],
      tableData: [
        {
          id: 1,
          percentual: 87,
          label: "Mandorle",
          icon: "mdi-account-circle",
          startDate: null,
          endDate: null,
          startDateMenu: false,
          endDateMenu: false,
        },
      ],
      selected: [],
      nextId: 2,
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),

      modal: false,
      menu2: false,
      firstName: "empty",
      date: "empty",
      Area: "",
      documents: "",
      Proposedsuppliers: "",
      selectedUser: null,
      userList: [],
      detailedUserData: null,
      type_expenditure: null,
      expenditureTypes: [],
      client_code: null,
      clientList: client,
      description_works: "",
      reference_document: "",
      extra_project: "",
      selectedGeneralname: null,
      selectedObject: null,
      projectList: this.flattenData(data),
      loading: false,
      error: null,
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
        this.type_expenditure &&
        ["Reimbursable_by_Client", "Backcharge"].includes(
          this.type_expenditure.name
        )
      );
    },
  },
  mounted() {
    this.fetchUsers();
    this.fetchExpenditure();
  },
  methods: {
    addRow() {
      this.tableData.push({
        id: this.nextId++,
        percentual: "",
        label: "",
        icon: "",
        startDate: null,
        endDate: null,
        startDateMenu: false,
        endDateMenu: false,
      });
    },
    deleteRow(index) {
      this.tableData.splice(index, 1);
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
  },
};
</script>
