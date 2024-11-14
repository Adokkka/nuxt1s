<template>
  <treeselect
    :multiple="true"
    :options="options"
    :disable-branch-nodes="true"
    v-model="value"
    search-nested
    :normalizer="normalizer"
    :show-count="true"
  />
</template>

<script>
import Vue from "vue";
import Vuex, { mapState, mapMutations } from "vuex";
import generateOptions from "../public/data/getservicestree.json";
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import service from "../public/data/getservicestree.json";

export default {
  components: { Treeselect },
  data: () => ({
    value: null,
    options: service,
    normalizer(node) {
      return {
        id: node.uid,
        label: node.name,
        children: node.items && node.items.length > 0 ? node.items : undefined,
      };
    },
  }),
};
</script>
