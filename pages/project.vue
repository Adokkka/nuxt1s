<template>
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
      <p><strong>Generalname:</strong> {{ selectedObject.Generalname }}</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import data from "../public/data/getcostcentertree.json";

export default {
  data() {
    return {
      selectedGeneralname: null,
      selectedObject: null,
      projectList: [],
    };
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
  methods: {
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
  },
};
</script>

<style scoped>
.object-details {
  margin-top: 20px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f9f9f9;
}
</style>
