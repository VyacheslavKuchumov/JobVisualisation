<template>
  <v-card
    width="1100"
    class="elevation-5 mt-5 ml-auto mr-auto"
    v-if="editedProject"
  >
    <v-table>
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title
            v-if="
              editedProject.shooting_start_date !==
              editedProject.shooting_end_date
            "
          >
            Оборудование в съёмке: "{{ editedProject.project_name }}" ({{
              new Date(editedProject.shooting_start_date).toLocaleDateString(
                "ru",
                {
                  year: "numeric",
                  month: "long",
                  day: "numeric",
                }
              )
            }}
            -
            {{
              new Date(editedProject.shooting_end_date).toLocaleDateString(
                "ru",
                {
                  year: "numeric",
                  month: "long",
                  day: "numeric",
                }
              )
            }}
            )
          </v-toolbar-title>
          <v-toolbar-title v-else>
            Оборудование в съёмке: "{{ editedProject.project_name }}" ({{
              new Date(editedProject.shooting_start_date).toLocaleDateString(
                "ru",
                {
                  year: "numeric",
                  month: "long",
                  day: "numeric",
                }
              )
            }})
          </v-toolbar-title>
        </v-toolbar>
      </template>
      <v-container>
        <v-row>
          <v-col cols="auto">
            <v-container width="500">
              <v-data-table
                v-if="equipment"
                :group-by="groupByAvailable"
                :headers="headers"
                :items="filteredEquipment"
                :items-per-page="-1"
                fixed-header
                hide-default-footer
              >
                <template v-slot:top>
                  <v-toolbar flat>
                    <v-toolbar-title>Доступное оборудование</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
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

                      <!-- Button to add the entire group -->
                      <v-btn
                        class="ml-5"
                        size="small"
                        color="blue-darken-1"
                        @click.stop="addGroup(item)"
                        v-if="
                          !editedProject.equipment.some(
                            (equip) =>
                              equip.equipment_set.equipment_set_name ===
                              item.value
                          ) && groupClassify(item) === 'second-group'
                        "
                      >
                        <v-icon>mdi-plus</v-icon>
                      </v-btn>
                    </td>
                  </tr>
                </template>

                <template v-slot:item.action_add="{ item }">
                  <v-btn
                    class="mr-5"
                    size="small"
                    color="blue-darken-1"
                    @click="addItem(item)"
                  >
                    <v-icon>mdi-plus</v-icon>
                  </v-btn>
                </template>
                <template v-slot:item.status="{ item }">
                  <v-icon
                    v-if="
                      availableEquipmentInOtherProjects.has(item.equipment_id)
                    "
                    color="blue"
                  >
                    mdi-camera
                  </v-icon>
                  <v-icon v-else color="green">mdi-check-circle</v-icon>
                </template>

                <template v-slot:no-data>Нет данных</template>
              </v-data-table>
            </v-container>
          </v-col>

          <v-col cols="auto">
            <v-container width="500">
              <v-data-table
                v-if="equipment"
                :group-by="groupByInProject"
                :headers="headers"
                :items="editedProject.equipment"
                :items-per-page="-1"
                fixed-header
                hide-default-footer
              >
                <template v-slot:item="{ item }">
                  <tr>
                    <td></td>
                    <td>{{ item.equipment_name }}</td>
                    <td>
                      <v-btn
                        class="mr-5"
                        size="small"
                        color="red-darken-1"
                        @click="deleteItem(item)"
                      >
                        <v-icon>mdi-delete</v-icon>
                      </v-btn>
                    </td>
                  </tr>
                </template>
                <template v-slot:top>
                  <v-toolbar flat>
                    <v-toolbar-title>В съёмке</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
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

                      <!-- Button to remove the entire group -->
                      <v-btn
                        class="ml-5"
                        size="small"
                        color="red-darken-1"
                        @click.stop="removeGroup(item)"
                        v-if="
                          editedProject.equipment.some(
                            (equip) =>
                              equip.equipment_set.equipment_set_name ===
                              item.value
                          )
                        "
                      >
                        <v-icon>mdi-delete</v-icon>
                      </v-btn>
                    </td>
                  </tr>
                </template>

                <template v-slot:item.action_delete="{ item }">
                  <v-btn
                    class="mr-5"
                    size="small"
                    color="red-darken-1"
                    @click="deleteItem(item)"
                  >
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </template>

                <template v-slot:no-data>Нет данных</template>
              </v-data-table>
            </v-container>
          </v-col>
        </v-row>
      </v-container>
    </v-table>
  </v-card>
</template>

<script>
import { mapActions, mapState } from "vuex";
import { useRoute } from "vue-router";

export default {
  data() {
    return {
      search: "",
    };
  },
  computed: {
    ...mapState("projects", ["editedProject", "projects"]),
    ...mapState("equipment", ["equipment"]),
    headers() {
      return [
        { title: "Статус", key: "status", sortable: false },
        { title: "Название", key: "equipment_name" },
        { title: "", key: "action_add", sortable: false },
        { title: "", key: "action_delete", sortable: false },
      ];
    },
    groupByAvailable() {
      return [
        // Define your grouping criteria here
        { key: "equipment_set.type.set_type_name", order: "asc" },
        { key: "equipment_set.equipment_set_name", order: "asc" },
      ];
    },
    groupByInProject() {
      return [
        // Define your grouping criteria here
        { key: "equipment_set.type.set_type_name", order: "asc" },
        { key: "equipment_set.equipment_set_name", order: "asc" },
      ];
    },
    filteredEquipment() {
      // Filter equipment that is not already in the edited project
      return this.equipment.filter(
        (equip) =>
          !this.editedProject.equipment.some(
            (projectEquip) => projectEquip.equipment_id === equip.equipment_id
          )
      );
    },
    availableEquipmentInOtherProjects() {
      if (!this.editedProject || !this.projects) return new Set();

      // Создаем объект для хранения оборудования, которое встречается в нескольких проектах
      const equipmentDateMap = {};

      this.projects.forEach((project) => {
        if (project.shooting_date === this.editedProject.shooting_date) {
          project.equipment.forEach((eq) => {
            if (!equipmentDateMap[eq.equipment_id]) {
              equipmentDateMap[eq.equipment_id] = new Set();
            }
            equipmentDateMap[eq.equipment_id].add(project.project_id);
          });
        }
      });

      const sharedEquipment = new Set();
      for (const [equipmentId, projectIds] of Object.entries(
        equipmentDateMap
      )) {
        sharedEquipment.add(equipmentId);
      }

      return sharedEquipment;
    },
  },
  methods: {
    ...mapActions("projects", [
      "getAllProjects",
      "getProjectByID",
      "updateProject",
      "addEquipmentToProject",
      "removeEquipmentFromProject",
    ]),
    ...mapActions("equipment", ["getAllEquipment"]),
    deleteItem(item) {
      const data = {
        project_id: this.editedProject.project_id,
        equipment_id: item.equipment_id,
      };
      this.removeEquipmentFromProject(data);
    },
    addItem(item) {
      const data = {
        project_id: this.editedProject.project_id,
        equipment_id: item.equipment_id,
      };
      this.addEquipmentToProject(data);
    },
    addGroup(group) {
      // Add all equipment items from the selected group to the project
      const equipmentToAdd = this.filteredEquipment.filter(
        (equip) => equip.equipment_set.equipment_set_name === group.value
      );
      equipmentToAdd.forEach((equip) => {
        const data = {
          project_id: this.editedProject.project_id,
          equipment_id: equip.equipment_id,
        };
        this.addEquipmentToProject(data);
      });
    },
    removeGroup(group) {
      // Remove all equipment items from the selected group from the project
      const equipmentToRemove = this.editedProject.equipment.filter(
        (equip) => equip.equipment_set.equipment_set_name === group.value
      );
      equipmentToRemove.forEach((equip) => {
        const data = {
          project_id: this.editedProject.project_id,
          equipment_id: equip.equipment_id,
        };
        this.removeEquipmentFromProject(data);
      });
    },
    groupClassify(item) {
      if (item.key === "equipment_set.type.set_type_name") {
        return "first-group";
      } // Adjust this condition
      if (item.key === "equipment_set.equipment_set_name") {
        return "second-group";
      }
    },
  },
  created() {
    const route = useRoute();
    const projectId = route.params.id;
    this.getAllProjects();
    this.getProjectByID(projectId);
    this.getAllEquipment();
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
