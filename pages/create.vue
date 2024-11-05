<template>
  <v-container>
    <v-row>
      <!-- Left Column: User Selection Field -->
      <v-col cols="4">
        <v-card>
          <v-card-title>User Selection</v-card-title>
          <v-card-text>
            <v-autocomplete
              max-height="300"
              v-model="selectedUser"
              :items="userList"
              item-text="full_name"
              item-value="id"
              label="Select or write user"
              return-object
              outlined
            ></v-autocomplete>
          </v-card-text>
        </v-card>
      </v-col>
      <v-spacer />
      <!-- Right Column: Project Selection Field -->
      <v-col cols="4" class="text-right">
        <v-card>
          <v-card-title>Project Selection</v-card-title>
          <v-card-text>
            <v-autocomplete
              v-model="selectedProject"
              :items="projectList"
              item-text="name"
              item-value="uid"
              label="Select Project"
              return-object
              outlined
            ></v-autocomplete>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Display selected user details -->
    <div v-if="selectedUser">
      <p><strong>Selected User Details:</strong></p>
      <p v-if="detailedUserData">
        IIN: {{ detailedUserData.IIN }}<br />
        Full Name: {{ detailedUserData.FIO }}<br />
        Organization: {{ detailedUserData.orgName }}<br />
        Department: {{ detailedUserData.depname }}<br />
        Position: {{ detailedUserData.Post }}
      </p>
      <p v-else>
        <em>No additional information available for this user.</em>
      </p>
    </div>

    <!-- Display selected project details -->
    <div v-if="selectedProject">
      <p><strong>Selected Project Details:</strong></p>
      <p>Code: {{ selectedProject.code }}</p>
      <p>Name: {{ selectedProject.name }}</p>
      <p>GeneralName: {{ selectedProject.Generalname }}</p>
    </div>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      selectedUser: null,
      selectedProject: null,
      userList: [], // List of users from the main API
      projectList: [], // List of projects from the project API
      detailedUserData: null, // Data from the detailed API call
    };
  },
  watch: {
    selectedUser(newUser) {
      if (newUser) {
        this.fetchDetailedUserData(newUser.username);
      } else {
        this.detailedUserData = null;
      }
    },
  },
  mounted() {
    this.fetchUsers();
    this.fetchProjects();
  },
  methods: {
    async fetchUsers() {
      try {
        const response = await axios.get("http://intra.isker.kz/api/v1/users");
        if (Array.isArray(response.data)) {
          this.userList = response.data.map((user) => ({
            ...user,
            full_name: `${user.last_name} ${user.first_name}`.trim(),
          }));
        }
      } catch (error) {
        console.error("Error fetching user list:", error);
      }
    },
    async fetchDetailedUserData(username) {
      try {
        const response = await axios.get(
          `http://192.168.0.67/api/v1/1c/websrf/getuser?iin=${username}`
        );
        this.detailedUserData = response.data.data || null;
      } catch (error) {
        if (error.response && error.response.status === 400) {
          console.warn("No data found for the provided IIN.");
          this.detailedUserData = null;
        } else {
          console.error("Error fetching detailed user data:", error.message);
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
        console.error("Error fetching project list:", error);
      }
    },
    flattenProjects(projects) {
      // Helper function to flatten nested project data
      const flatList = [];
      function flatten(items) {
        items.forEach((item) => {
          flatList.push({
            code: item.code,
            name: item.name,
            uid: item.uid,
            Generalname: item.Generalname,
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
};
</script>

<style scoped>
/* Add any custom styles here */
</style>
