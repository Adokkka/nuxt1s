<template>
  <v-row justify="center">
    <!-- Display selected item outside the dialog -->
    <v-col cols="12" sm="6">
      <div v-if="selectedItem" type="info">
        Selected Item: {{ selectedItem.name }}
      </div>
      <div v-else>Hi</div>
    </v-col>

    <!-- Dialog for user profile -->
    <v-dialog v-model="dialog" persistent max-width="600px">
      <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark v-bind="attrs" v-on="on">
          Open Dialog
        </v-btn>
      </template>
      <v-card>
        <v-card-title>
          <span class="text-h5">User Profile</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" sm="6">
                <v-treeview
                  v-model="selectedItems"
                  selectable
                  :multiple="false"
                  disable-branch-nodes
                  item-children="items"
                  item-key="uid"
                  :items="items"
                  selection-type="leaf"
                  disable-per-node
                  activatable
                ></v-treeview>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="dialog = false">
            Close
          </v-btn>
          <v-btn color="blue darken-1" text @click="dialog = false">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>
<script>
import servisetree from "../public/data/getservicestree.json";

export default {
  data: () => ({
    dialog: false,
    items: servisetree,
    selectedItems: [], // Array to hold selected items' keys
  }),
  computed: {
    // Get the selected item object based on the selected UID
    selectedItem() {
      if (this.selectedItems.length > 0) {
        const selectedItemKey = this.selectedItems[0]; // Get the first selected key
        return this.findItemByKey(selectedItemKey);
      }
      return null;
    },
  },
  methods: {
    // Recursive function to find the item based on its UID
    findItemByKey(uid) {
      function findRecursive(items) {
        for (let item of items) {
          if (item.uid === uid) {
            return item;
          }
          if (item.items) {
            const found = findRecursive(item.items);
            if (found) {
              return found;
            }
          }
        }
        return null;
      }
      return findRecursive(this.items);
    },
  },
};
</script>
