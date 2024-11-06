<template>
  <v-container>
    <v-row margin="100">
      <v-col>
        <h1>WebSRF_FORM</h1>
      </v-col>
      <v-sheet class="mx-auto" width="300" color="#121212">
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
          <p>Position: {{ detailedUserData.Post }}</p>
          <p>Department: {{ detailedUserData.depname }}</p>
        </div>
        <div v-else>
          <p>
            <em>No additional information available for this user.</em>
          </p>
        </div>
      </v-col>
      <v-col cols="6" class="text-right">
        <!-- Autocomplete by Project Name -->
        <v-autocomplete
          v-model="selectedProjectByName"
          :items="projectList"
          item-text="name"
          item-value="uid"
          label="Search by Project Name"
          return-object
          outlined
          @change="handleProjectNameChange"
        ></v-autocomplete>

        <!-- Autocomplete by Project Code (auto-fills when project name is selected) -->
        <v-autocomplete
          v-model="selectedProjectByCode"
          :items="projectList"
          item-text="code"
          item-value="uid"
          label="Search by Project Code"
          return-object
          outlined
          @change="handleProjectCodeChange"
        ></v-autocomplete>

        <!-- Display selected project details -->
        <div v-if="selectedProjectDetails">
          <p><strong>Selected Project Details:</strong></p>
          <p>Code: {{ selectedProjectDetails.code }}</p>
          <p>Name: {{ selectedProjectDetails.name }}</p>
          <p>UID: {{ selectedProjectDetails.uid }}</p>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      firstName: "adilet",
      date: "2024/08/01/11:39:30",
      selectedProjectByName: null,
      selectedProjectByCode: null,
      selectedProjectDetails: null,
      projectList: [],
      selectedUser: null,
      userList: [],
      detailedUserData: null,
    };
  },
  mounted() {
    this.fetchProjects();
    this.fetchUsers();
  },
  methods: {
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
    handleProjectNameChange(selectedProject) {
      if (selectedProject) {
        this.selectedProjectDetails = selectedProject;
        // Set the selected project by code based on name selection
        this.selectedProjectByCode = this.projectList.find(
          (project) => project.uid === selectedProject.uid
        );
      } else {
        // Reset both fields if no project is selected by name
        this.selectedProjectByCode = null;
        this.selectedProjectDetails = null;
      }
    },
    handleProjectCodeChange(selectedProject) {
      if (selectedProject) {
        this.selectedProjectDetails = selectedProject;
        // Set the selected project by name based on code selection
        this.selectedProjectByName = this.projectList.find(
          (project) => project.uid === selectedProject.uid
        );
      } else {
        // Reset both fields if no project is selected by code
        this.selectedProjectByName = null;
        this.selectedProjectDetails = null;
      }
    },
  },
};
</script>

<style scoped>
/* Add any custom styles here */
</style>
