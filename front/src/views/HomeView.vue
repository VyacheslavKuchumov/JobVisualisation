<template>
  <v-card class="elevation-5 mt-5 ml-auto mr-auto" width="1100">
    <v-data-table
      :headers="headers"
      :items="jobs"
      :group-by="groupBy"
      :items-per-page="-1"
      hide-default-footer
      fixed-header
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Данные с hh.ru</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>

          <!-- Upload Button -->
          <v-btn class="mb-2" color="primary" dark @click="triggerFileInput">
            Загрузить CSV
          </v-btn>
          <input
            ref="fileInput"
            type="file"
            accept=".csv"
            @change="handleFileUpload"
            style="display: none"
          />
        </v-toolbar>
      </template>

      <template
                  v-slot:group-header="{
                    item,
                    columns,
                    toggleGroup,
                    isGroupOpen,
                  }"
                >
                  <tr>
                    <td :colspan="columns.length" @click="toggleGroup(item)">
                      <v-btn
                        :class="groupClassify(item)"
                        :icon="
                          isGroupOpen(item)
                            ? 'mdi-chevron-down'
                            : 'mdi-chevron-right'
                        "
                        size="small"
                        variant="text"
                      ></v-btn>
                      <span :class="groupClassify(item)">{{ item.value }}</span>

                      
                    </td>
                  </tr>
                </template>

      <template v-slot:no-data> Нет данных </template>
    </v-data-table>
  </v-card>
</template>

<script>
import Papa from "papaparse"; // Ensure you install papaparse with `npm install papaparse`

export default {
  name: "HomeView",
  data() {
    return {
      jobs: [], // Parsed CSV data
    };
  },
  computed: {
    headers() {
      // Define headers for your table
      return [
        { title: "Дата ввода", key: "entry_date", sortable: true },
        { title: "Локация", key: "area_name", sortable: true },
        { title: "Проф роль", key: "professional_roles_name", sortable: true },
        { title: "Группа вакансий", key: "job_group", sortable: true },
        { title: "Количество", key: "Count(url)", sortable: true },
      ];
    },
    groupBy() {
      return [
        // Define your grouping criteria here
        { key: "area_name", order: "asc" },
        { key: "entry_date", order: "asc"},
        { key: "job_group", order: "asc" },
      ];
    },
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.click(); // Simulate file input click
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        Papa.parse(file, {
          header: true, // Automatically set headers based on CSV
          skipEmptyLines: true, // Skip empty rows
          complete: (result) => {
            this.jobs = result.data; // Assign parsed data to jobs
          },
          error: (error) => {
            console.error("CSV Parsing Error:", error); // Log errors
          },
        });
      }
    },
    groupClassify(item) {
      if (item.key === "area_name") {
        return "first-group";
      } // Adjust this condition
      if (item.key === "job_group") {
        return "second-group";
      }
    },
  },
};
</script>

<style scoped>
.second-group {
  margin-left: 20px; /* Adjust this value for more or less indentation */
}
.first-group {
  font-weight: bolder; /* Adjust this value for more or less indentation */
}
</style>