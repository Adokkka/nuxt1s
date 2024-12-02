<template>
  <div>
    <p>Generated UUID: {{ uuid }}</p>
    <button @click="generateUuid">Generate UUID</button>
  </div>
</template>

<script>
//import { v4 as uuidv4 } from "uuid";

export default {
  data() {
    return {
      uuid: "",
    };
  },
  methods: {
    generateUuid() {
      // this.uuid = uuidv4(); // Generate a new UUID and assign it to data
    },
  },
};
</script>
