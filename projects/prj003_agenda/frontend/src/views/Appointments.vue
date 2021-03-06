<template>
  <v-data-table
    :headers="headers"
    :items="appointments"
    :items-per-page="3"
    sort-by="begins_at_date"
    class="elevation-1"
    :footer-props="{
      showFirstLastPage: true,
      firstIcon: 'mdi-arrow-collapse-left',
      lastIcon: 'mdi-arrow-collapse-right',
      prevIcon: 'mdi-minus',
      nextIcon: 'mdi-plus',
      itemsPerPageText: 'Linhas por página',
      itemsPerPageOptions: [3, 6, 9, -1]
    }"
  >
    <template v-slot:[`item.begins_at_date`]="{ item }">
      <span>{{ formatDate(item.begins_at_date) }}</span>
    </template>
    <template v-slot:[`item.begins_at_time`]="{ item }">
      <span>{{ formatTime(item.begins_at_time) }}</span>
    </template>
    <template v-slot:[`item.ends_at_date`]="{ item }">
      <span>{{ formatDate(item.ends_at_date) }}</span>
    </template>
    <template v-slot:[`item.ends_at_time`]="{ item }">
      <span>{{ formatTime(item.ends_at_time) }}</span>
    </template>
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Eventos</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              Novo evento
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.title"
                      label="Nome"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.description"
                      label="descrição"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      ref="dt_ini_menu"
                      v-model="dt_ini_menu"
                      :close-on-content-click="false"
                      :return-value.sync="editedItem.begins_at_date"
                      transition="scale-transition"
                      offset-y
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="formattedBeginDate"
                          label="Data de inicio"
                          prepend-icon="mdi-calendar"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="editedItem.begins_at_date"
                        locale="pt-br"
                        scrollable
                      >
                        <v-spacer></v-spacer>
                        <v-btn
                          text
                          color="primary"
                          @click="dt_ini_menu = false"
                        >
                          Cancelar
                        </v-btn>
                        <v-btn
                          text
                          color="primary"
                          @click="
                            $refs.dt_ini_menu.save(editedItem.begins_at_date)
                          "
                        >
                          OK
                        </v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      ref="tm_ini_menu"
                      v-model="tm_ini_menu"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      :return-value.sync="editedItem.begins_at_time"
                      transition="scale-transition"
                      offset-y
                      max-width="290px"
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="editedItem.begins_at_time"
                          label="hora"
                          prepend-icon="mdi-clock-time-four-outline"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-time-picker
                        v-if="tm_ini_menu"
                        v-model="editedItem.begins_at_time"
                        format="24hr"
                        @click:minute="
                          $refs.tm_ini_menu.save(editedItem.begins_at_time)
                        "
                      ></v-time-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      ref="dt_fim_menu"
                      v-model="dt_fim_menu"
                      :close-on-content-click="false"
                      :return-value.sync="editedItem.ends_at_date"
                      transition="scale-transition"
                      offset-y
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="formattedEndDate"
                          label="Data do fim"
                          prepend-icon="mdi-calendar"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="editedItem.ends_at_date"
                        locale="pt-br"
                        scrollable
                      >
                        <v-spacer></v-spacer>
                        <v-btn
                          text
                          color="primary"
                          @click="dt_fim_menu = false"
                        >
                          Cancelar
                        </v-btn>
                        <v-btn
                          text
                          color="primary"
                          @click="
                            $refs.dt_fim_menu.save(editedItem.ends_at_date)
                          "
                        >
                          OK
                        </v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      ref="tm_fim_menu"
                      v-model="tm_fim_menu"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      :return-value.sync="editedItem.ends_at_time"
                      transition="scale-transition"
                      offset-y
                      max-width="290px"
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="editedItem.ends_at_time"
                          label="hora"
                          prepend-icon="mdi-clock-time-four-outline"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-time-picker
                        v-if="tm_fim_menu"
                        v-model="editedItem.ends_at_time"
                        format="24hr"
                        @click:minute="
                          $refs.tm_fim_menu.save(editedItem.ends_at_time)
                        "
                      ></v-time-picker>
                    </v-menu>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">
                Cancelar
              </v-btn>
              <v-btn color="blue darken-1" text @click="save">
                Salvar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline"
              >Tem certeza que quer eliminar o evento?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancelar</v-btn
              >
              <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:[`item.actions`]="{ item }">
      <!-- <template v-slot:item.actions="{ item }"> -->
      <v-icon small class="mr-2" @click="editItem(item)">
        mdi-pencil
      </v-icon>
      <v-icon small @click="deleteItem(item)">
        mdi-delete
      </v-icon>
      <v-icon small @click="startMeeting(item)">
        mdi-account-multiple-plus
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">
        Reset
      </v-btn>
    </template>
  </v-data-table>
</template>

<script>
import { mapGetters } from "vuex";
import { mapActions } from "vuex";
import moment from "moment";

export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    dt_ini_menu: false,
    tm_ini_menu: false,
    dt_fim_menu: false,
    tm_fim_menu: false,
    headers: [
      {
        text: "Nome",
        align: "start",
        sortable: false,
        value: "title"
      },
      { text: "descrição", value: "description" },
      { text: "inicio", value: "begins_at_date" },
      { text: "hora", value: "begins_at_time" },
      { text: "fim", value: "ends_at_date" },
      { text: "hora", value: "ends_at_time" },
      { text: "Ações", value: "actions", sortable: false }
    ],
    appointments: [],
    editedIndex: -1,
    editedItem: {
      title: "",
      description: "",
      begins_at_date: "",
      begins_at_time: "",
      ends_at_date: "",
      ends_at_time: ""
    },
    defaultItem: {
      title: "",
      description: "",
      begins_at_date: "",
      begins_at_time: "",
      ends_at_date: "",
      ends_at_time: ""
    }
  }),

  computed: {
    ...mapGetters(["authentication"]),
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
    formattedBeginDate() {
      return this.editedItem.begins_at_date
        ? this.formatDate(this.editedItem.begins_at_date)
        : "";
    },
    // formattedBeginTime() {
    //   return this.editedItem.begins_at_time
    //     ? this.formatTime(this.editedItem.begins_at_time)
    //     : "";
    // },
    formattedEndDate() {
      return this.editedItem.ends_at_date
        ? this.formatDate(this.editedItem.ends_at_date)
        : "";
    }
    // formattedEndTime() {
    //   return this.editedItem.ends_at_time
    //     ? this.formatTime(this.editedItem.ends_at_time)
    //     : "";
    // }
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    }
  },

  created() {
    this.initialize();
  },

  methods: {
    ...mapActions(["set_active_meeting"]),

    formatDate(str) {
      const result = moment(str)
        .locale("pt-br")
        .format("ddd, DD/MM/YYYY");
      return result;
    },
    formatTime(str) {
      const result = moment(str, "HH:mm:ss").format("HH:mm");
      return result;
    },
    initialize() {
      this.getAppointments().then(
        () => {
          console.log("Compromissos carregados!");
        },
        error => console.log("Erro carregando os compromissos: ", error)
      );
    },

    editItem(item) {
      this.editedIndex = this.appointments.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.appointments.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.appointments.splice(this.editedIndex, 1);
      this.removeAppointment(this.editedItem);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.appointments[this.editedIndex], this.editedItem);
        this.updateAppointment(this.editedItem);
      } else {
        this.appointments.push(this.editedItem);
        this.createAppointment(this.editedItem);
      }
      console.log(">>> Em save() - editedItem:");
      console.log(
        "   id: ",
        this.editedItem.id ? this.editedItem.id : "Não há"
      );
      console.log("   nome: ", this.editedItem.title);
      this.close();
    },

    getAppointments() {
      return new Promise((resolve, reject) => {
        console.log(
          "Em getAppointments, authentication: ",
          `${this.authentication}`
        );
        // .get("http://127.0.0.1:8000/api/appointments/", {
        this.$http
          .get("api/appointments/", {
            headers: {
              Authorization: `${this.authentication}`
            }
          })
          .then(response => response.json())
          .then(
            response => {
              console.log("response: ", response);
              this.appointments = response;
              this.appointments.forEach(appointment => {
                console.log(
                  "appointment - id: ",
                  appointment.id,
                  " user: ",
                  appointment.user,
                  " title: ",
                  appointment.title,
                  " begin date: ",
                  appointment.begins_at_date,
                  " begin time: ",
                  appointment.begins_at_time,
                  " end date: ",
                  appointment.ends_at_date,
                  " end time: ",
                  appointment.ends_at_time
                );
                appointment.begins_at_time = appointment.begins_at_time
                  ? appointment.begins_at_time.substring(0, 5)
                  : null;
                appointment.ends_at_time = appointment.ends_at_time
                  ? appointment.ends_at_time.substring(0, 5)
                  : null;
              });
              resolve(response);
            },
            error => {
              if (error.status == 400) {
                alert(
                  "Unable to get appointments. " +
                    JSON.stringify(error.bodyText).replace(/[\\,,{,}"]/g, "")
                );
              } else {
                alert("We are sorry, there was a failure. Try again latter!");
              }
              console.log("Error: ", error);
              reject(error);
            }
          );
      });
    },
    createAppointment(payload) {
      return new Promise((resolve, reject) => {
        console.log(
          "Em createAppointment, authentication: ",
          `${this.authentication}`
        );
        // .get("http://127.0.0.1:8000/api/appointments/", {
        this.$http
          .post("api/appointments/", JSON.stringify(payload), {
            headers: {
              Authorization: `${this.authentication}`
            }
          })
          .then(response => response.json())
          .then(
            response => {
              console.log("response: ", response);
              resolve(response);
            },
            error => {
              console.log("Error: ", error);
              reject(error);
            }
          );
      });
    },
    updateAppointment(payload) {
      return new Promise((resolve, reject) => {
        console.log(
          "Em updateAppointment, authentication: ",
          `${this.authentication}`
        );
        // .get("http://127.0.0.1:8000/api/appointments/", {
        this.$http
          .put(`api/appointments/${payload.id}/`, JSON.stringify(payload), {
            headers: {
              Authorization: `${this.authentication}`
            }
          })
          .then(response => response.json())
          .then(
            response => {
              console.log("response: ", response);
              resolve(response);
            },
            error => {
              console.log("Error: ", error);
              reject(error);
            }
          );
      });
    },
    removeAppointment(payload) {
      return new Promise((resolve, reject) => {
        console.log(
          "Em removeAppointment, authentication: ",
          `${this.authentication}`
        );
        // .get("http://127.0.0.1:8000/api/appointments/", {
        this.$http
          .delete(`api/appointments/${payload.id}/`, {
            headers: {
              Authorization: `${this.authentication}`
            }
          })
          .then(
            response => {
              console.log("response: ", response);
              console.log("response status: ", response.status);
              resolve(response);
            },
            error => {
              console.log("Error: ", error);
              reject(error);
            }
          );
      });
    },
    startMeeting() {
      this.set_active_meeting(true);
    }
  }
};
</script>
