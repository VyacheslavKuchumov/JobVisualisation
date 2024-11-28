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

      <!-- Group Header with Count -->
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
              :icon="isGroupOpen(item) ? 'mdi-chevron-down' : 'mdi-chevron-right'"
              size="small"
              variant="text"
            ></v-btn>
            <span :class="groupClassify(item)">{{ item.value }}</span>
            <span class="group-count">({{ getGroupCount(item) }})</span>
          </td>
        </tr>
      </template>

      <template v-slot:no-data> Нет данных </template>
    </v-data-table>
  </v-card>
</template>

<script>
import Papa from "papaparse";

export default {
  name: "HomeView",
  data() {
    return {
      jobs: [], // Parsed CSV data
    };
  },
  computed: {
    headers() {
      return [
        
        
        { title: "Проф роль", key: "professional_roles_name", sortable: true },
        
        { title: "Количество", key: "Count(url)", sortable: true },
      ];
    },
    groupBy() {
      return [
        { key: "area_name", order: "asc" },
        { key: "entry_date", order: "asc" },
        { key: "job_group", order: "asc" },
      ];
    },
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        Papa.parse(file, {
          header: true,
          skipEmptyLines: true,
          complete: (result) => {
            this.jobs = result.data.map((job) => ({
              ...job,
              "Count(url)": parseInt(job["Count(url)"], 10) || 0,
            }));
          },
          error: (error) => {
            console.error("CSV Parsing Error:", error);
          },
        });
      }
    },
    groupClassify(item) {
      if (item.key === "area_name") {
        return "first-group";
      }
      if (item.key === "job_group") {
        return "second-group";
      }
    },
    getGroupCount(item) {
      const groupItems = this.jobs.filter((job) => job[item.key] === item.value);
      return groupItems.reduce((sum, job) => sum + (job["Count(url)"] || 0), 0);
    },
  },
};
</script>

<style scoped>
.second-group {
  margin-left: 20px;
}
.first-group {
  font-weight: bolder;
}
.group-count {
  margin-left: 8px;
  color: gray;
  font-style: italic;
}
</style>
