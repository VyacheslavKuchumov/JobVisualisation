<template>
  <v-container width="600">
    <v-row>
      <v-col cols="12">
        <v-data-table
          :headers="headers"
          :items="Warehouses"
          item-value="id"
          class="elevation-1"
          style="table-layout: auto; width: auto"
          :items-per-page="-1"
          fixed-header
          hide-default-footer
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-toolbar-title>Склады</v-toolbar-title>
              <v-divider class="mx-4" inset vertical></v-divider>
              <v-spacer></v-spacer>
              <v-btn class="mb-2" color="primary" dark @click="openDialog()">
                Добавить
              </v-btn>
            </v-toolbar>
          </template>

          <template v-slot:item.actions_edit="{ item }">
            <v-btn size="small" color="blue-darken-1" @click="openDialog(item)">
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
          </template>

          <template v-slot:item.actions_delete="{ item }">
            <v-btn
              size="small"
              color="red-darken-1"
              @click="deleteItem(item.warehouse_id)"
            >
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </template>
        </v-data-table>
      </v-col>
    </v-row>

    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline"
            >{{ form.warehouse_id ? "Изменить" : "Добавить" }} склад</span
          >
        </v-card-title>
        <v-card-text>
          <v-text-field
            v-model="form.warehouse_name"
            label="Warehouse Name"
            required
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="closeDialog()">Cancel</v-btn>
          <v-btn color="primary" @click="saveItem()">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { mapState, mapActions } from "vuex";

export default {
  data() {
    return {
      dialog: false,
      form: { warehouse_id: null, warehouse_name: "" },
      headers: [
        { title: "Name", value: "warehouse_name", width: "auto" },
        {
          title: "Изменить",
          value: "actions_edit",
          sortable: false,
          width: "20",
        },
        {
          title: "Удалить",
          value: "actions_delete",
          sortable: false,
          width: "20",
        },
      ],
    };
  },
  computed: {
    ...mapState("warehouse", ["warehouses"]),
    Warehouses() {
      return this.warehouses || [];
    },
  },
  methods: {
    ...mapActions("warehouse", [
      "getAllWarehouses",
      "createWarehouse",
      "updateWarehouse",
      "deleteWarehouse",
    ]),

    openDialog(item = null) {
      this.form = item
        ? { ...item }
        : { warehouse_id: null, warehouse_name: "" };
      this.dialog = true;
    },
    closeDialog() {
      this.dialog = false;
    },
    async saveItem() {
      if (this.form.warehouse_id) {
        await this.updateWarehouse(this.form);
      } else {
        await this.createWarehouse(this.form);
      }
      this.getAllWarehouses();
      this.closeDialog();
    },
    async deleteItem(id) {
      await this.deleteWarehouse({ warehouse_id: id });
      this.getAllWarehouses();
    },
  },
  beforeMount() {
    this.getAllWarehouses();
  },
};
</script>
