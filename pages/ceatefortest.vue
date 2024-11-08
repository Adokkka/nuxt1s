<template>
  <v-container>
    <v-row margin="100">
      <v-col>
        <h1>WebSRF_FORM</h1>
      </v-col>
      <v-sheet class="mx-auto" width="300">
        <v-form disabled>
          <v-text-field v-model="firstName" label="WEBSRF NO."></v-text-field>
          <v-text-field v-model="date" label="WEBSRF date."></v-text-field>
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
          <p>Position: {{ selectedUser.phone }}</p>
        </div>
        <div v-else>
          <em>No additional information available for this user.</em>
        </div>
      </v-col>
      <v-col cols="6" class="text-right">
        <div>
          <!-- Search by Generalname -->
          <v-autocomplete
            v-model="selectedGeneralname"
            :items="uniqueGeneralnames"
            label="Search by Generalname"
            outlined
            @change="resetSelection"
            required
          ></v-autocomplete>

          <!-- Search objects related to the selected Generalname -->
          <v-autocomplete
            v-model="selectedObject"
            :items="filteredObjects"
            item-text="name"
            item-value="uid"
            label="Search Related Objects"
            return-object
            outlined
          ></v-autocomplete>

          <!-- Display selected object details -->
          <div v-if="selectedObject" class="object-details">
            <h3>Selected Object Details:</h3>
            <p><strong>Name:</strong> {{ selectedObject.name }}</p>
            <p><strong>Code:</strong> {{ selectedObject.code }}</p>
            <p><strong>UID:</strong> {{ selectedObject.uid }}</p>
            <p>
              <strong>Generalname:</strong> {{ selectedObject.Generalname }}
            </p>
          </div>
        </div>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-sheet class="mx-auto">
          <v-form>
            <v-textarea
              v-model="Proposedsuppliers"
              label="Предлагаемые поставщики:"
              outlined
              row-height="8"
              auto-grow
              clearable
            ></v-textarea>
            <v-textarea
              v-model="Area"
              label="Участок:"
              outlined
              auto-grow
              row-height="8"
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
          row-height="8"
          @change="fetchExpenditure"
        ></v-autocomplete>
        <div
          v-if="
            (type_expenditure &&
              type_expenditure.name === 'Reimbursable_by_Client') ||
            (type_expenditure && type_expenditure.name === 'Backcharge')
          "
        >
          <v-autocomplete
            item-text="name"
            return-object
            outlined
            :items="clientList"
            @change="fetchClient"
            v-model="client_code"
            label="Клиент"
            item-value="bin"
            required
          ></v-autocomplete>
          <v-textarea
            v-model="Area"
            label="Ссылочный документ:"
            outlined
            auto-grow
            row-height="8"
            clearable
          ></v-textarea>
          <div v-if="client_code">
            <p><strong>Selected client Details:</strong></p>
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
        <V-textarea
          v-model="description_works"
          label="Описание работ"
          auto-grow
          outlined
        >
        </V-textarea
      ></v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import data from "../public/data/getcostcentertree.json";
export default {
  data() {
    return {
      firstName: "empty",
      date: "empty",
      Area: "",
      Proposedsuppliers: "",
      selectedProjectByName: null,
      selectedProjectByCode: null,
      selectedProjectDetails: null,

      selectedUser: null,
      userList: [],
      detailedUserData: null,
      type_expenditure: null,
      expenditureTypes: [],
      client_code: null,
      clientList: [],
      radios: "",
      description_works: "",
      currency: "USD",
      reference_document: "",
      extra_project: "",
      selectedGeneralname: null,
      selectedObject: null,
      projectList: [],
    };
  },
  mounted() {
    this.fetchProjects();
    this.fetchUsers();
    this.fetchExpenditure();
    this.fetchClient();
  },
  methods: {
    async fetchExpenditure() {
      try {
        const response = await axios.get(
          "http://192.168.0.67/api/v1/websrf/getexpenditure"
        );
        console.log("fetched data", response.data);
        if (Array.isArray(response.data)) {
          this.expenditureTypes = response.data.map((expenditure) => ({
            ...expenditure,
          }));
        } else {
          console.log("Unexpected data structure:", response.data);
        }
      } catch (error) {
        console.error(
          "Error fetching user data:",
          error.response ? error.response.data : error
        );
      }
    },
    resetSelection() {
      this.selectedObject = null;
    },
    flattenData(data) {
      return data.reduce((acc, item) => {
        acc.push(item);
        if (item.items?.length) {
          acc.push(...this.flattenData(item.items));
        }
        return acc;
      }, []);
    },
    async fetchClient() {
      try {
        const response = await axios.get(
          "http://1c-main/testIvan/hs/intra/getclientlist"
        );
        console.log("fetched data", response.data);
        if (Array.isArray(response.data)) {
          this.clientList = response.data.map((client) => ({
            ...client,
          }));
        } else {
          console.log("Unexpected data structure:", response.data);
        }
      } catch (error) {
        console.error(
          "Error fetching user data:",
          error.response ? error.response.data : error
        );
      }
    },
    async fetchUsers() {
      try {
        const response = await axios.get("http://intra.isker.kz/api/v1/users");
        console.log("fetched data", response.data);
        if (Array.isArray(response.data)) {
          this.userList = response.data.map((user) => ({
            ...user,
            full_name: `${user.last_name} ${user.first_name}`.trim(),
          }));
        } else {
          console.log("Unexpected data structure:", response.data);
        }
      } catch (error) {
        console.error(
          "Error fetching user data:",
          error.response ? error.response.data : error
        );
      }
    },
    async handleUserSelection(user) {
      if (user && user.username) {
        try {
          const response = await axios.get(
            `http://192.168.0.67/api/v1/1c/websrf/getuser?iin=${user.username}`
          );
          console.log("Detailed user data:", response.data);
          if (response.data && response.data.data) {
            this.detailedUserData = response.data.data;
          } else {
            console.error("Unexpected detailed data structure:", response.data);
          }
        } catch (error) {
          if (error.response && error.response.status === 400) {
            console.warn("No data found for the provided IIN.");
            this.detailedUserData = null; // Or show a specific message in the UI
          } else {
            console.error("Error fetching detailed user data:", error.message);
          }
        }
      }
    },
    async fetchProjects() {
      try {
        const response = await axios.get(
          "http://1c-main/testIvan/hs/intra/getcostcentertree"
        );
        if (Array.isArray(response.data)) {
          this.projectList = this.flattenProjects(response.data);
        }
      } catch (error) {
        console.error("Error fetching project data:", error);
      }
    },
    flattenProjects(projects) {
      // Flatten nested project data for easier access in autocompletes
      const flatList = [];
      function flatten(items) {
        items.forEach((item) => {
          flatList.push({
            code: item.code,
            name: item.name,
            uid: item.uid,
          });
          if (item.items && item.items.length > 0) {
            flatten(item.items);
          }
        });
      }
      flatten(projects);
      return flatList;
    },
  },
  computed: {
    uniqueGeneralnames() {
      return Array.from(
        new Set(this.projectList.map((item) => item.Generalname))
      );
    },
    filteredObjects() {
      return this.projectList.filter(
        (item) => item.Generalname === this.selectedGeneralname
      );
    },
  },
  async mounted() {
    try {
      this.projectList = this.flattenData(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  },
};
</script>

<style scoped>
/* Add any custom styles here */
</style>
