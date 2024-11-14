<template>
  <div id="app">
    <treeselect
      v-model="value"
      :multiple="true"
      :options="options"
      :normalizer="normalizer"
      :load-options="loadOptions"
      :show-count="true"
    />
  </div>
</template>

<script>
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import service from "../public/data/getservicestree.json";

export default {
  components: { Treeselect },
  data() {
    return {
      value: [],
      options: service,
    };
  },
  methods: {
    normalizer(node) {
      return {
        id: node.uid,
        label: node.name,
        children:
          node.groupe !== false && node.items && node.items.length > 0
            ? node.items
            : null, // Если items пуст, узел не ветка
        // isDisabled: node.groupe === true, // Отключить узлы с groupe: true
      };
    },
    async loadOptions({ action, parentNode }) {
      if (action === "LOAD_CHILDREN_OPTIONS") {
        // Пример: асинхронная загрузка данных
        const children = await fetchChildren(parentNode.id); // Замените fetchChildren на свой метод
        return children;
      }
    },
  },
};
</script>
