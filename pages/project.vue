<template>
  <v-container>
    <!-- Main Project Selection -->
    <v-autocomplete
      v-model="selectedProject"
      :items="projectList"
      item-text="name"
      item-value="uid"
      label="Select Project"
      return-object
      outlined
      @change="handleProjectChange"
    ></v-autocomplete>

    <!-- Display main project details -->
    <div v-if="selectedProject">
      <p><strong>Main Project Details:</strong></p>
      <p>Code: {{ selectedProject.code }}</p>
      <p>Name: {{ selectedProject.name }}</p>
    </div>

    <!-- Dynamic nested project selection -->
    <div v-for="(subProject, index) in selectedSubProjects" :key="index">
      <v-autocomplete
        v-model="subProject.selected"
        :items="subProject.items"
        item-text="name"
        item-value="uid"
        label="Select Subproject"
        return-object
        outlined
        @change="handleSubProjectChange(index)"
      ></v-autocomplete>

      <div v-if="subProject.selected">
        <p>
          <strong>Subproject Level {{ index + 1 }} Details:</strong>
        </p>
        <p>Code: {{ subProject.selected.code }}</p>
        <p>Name: {{ subProject.selected.name }}</p>
      </div>
    </div>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      selectedProject: null,
      projectList: [],
      selectedSubProjects: [], // Dynamic list for nested subproject selections
    };
  },
  mounted() {
    this.fetchProjects();
  },
  methods: {
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
      // Helper function to flatten the main project list
      const flatList = [];
      function flatten(items) {
        items.forEach((item) => {
          flatList.push({
            code: item.code,
            name: item.name,
            uid: item.uid,
            items: item.items || [],
            itemscol: item.itemscol || 0,
          });
          if (item.items && item.items.length > 0) {
            flatten(item.items);
          }
        });
      }
      flatten(projects);
      return flatList;
    },
    handleProjectChange(project) {
      // Clear previous subprojects when a new main project is selected
      this.selectedSubProjects = [];
      if (project && project.itemscol > 0) {
        // Add initial subproject list if itemscol > 0
        this.selectedSubProjects.push({
          items: project.items,
          selected: null,
        });
      }
    },
    handleSubProjectChange(index) {
      const selectedSubProject = this.selectedSubProjects[index].selected;
      // Clear all sublevels below the current index
      this.selectedSubProjects.splice(index + 1);
      // If the selected subproject has further nested items, add a new selection level
      if (selectedSubProject && selectedSubProject.itemscol > 0) {
        this.selectedSubProjects.push({
          items: selectedSubProject.items,
          selected: null,
        });
      }
    },
  },
};
</script>

<style scoped>
/* Add any custom styles here */
</style>
