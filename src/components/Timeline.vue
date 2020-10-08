<template>
  <div>
    <v-container>
      <!-- Card Timeline-->
      <h1>Agenda de reservas</h1>
      <v-card class="mx-auto" elevation="3" outlined>
        <v-card-text>
          <!--Timeline-->
          <div class="mx-10" id="visualization">
            <div class="menu">
              <input
                style="display: none"
                id="sliderZoom"
                type="range"
                class="range"
                name="a"
                min="-1"
                max="1"
                step="0.1"
                value="0"
              />
            </div>
          </div>
          <v-btn @click="moveLeft()" color="secondary" class="arrow-left">
            <v-icon class="white--text">mdi-chevron-left</v-icon>
          </v-btn>
          <v-btn @click="moveRight()" color="secondary" class="arrow-right">
            <v-icon class="white--text">mdi-chevron-right</v-icon>
          </v-btn>
          <!-- End Timeline -->
        </v-card-text>
      </v-card>
      <!-- End card timeline -->

      <!-- Table -->
      <v-data-table
        :headers="headers"
        :items="arreglo"
        sort-by="calories"
        class="elevation-1 mt-10"
        :hide-default-footer="true"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>Lista de reservas</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                  Agregar reserva
                </v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline mb-1">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                      <v-text-field
                        
                        class="mt-3 elevation-0"
                        outlined
                        v-model="editedItem.content"
                        label="Nombre"
                      ></v-text-field>
                      <!-- Date picker -->
                      <date-picker
                        class="ma-2"
                        v-model="editedItem.start"
                        type="datetime"
                        format="YYYY-MM-DD HH:mm"
                        width="400"
                        placeholder="Desde"
                      ></date-picker>
                      <date-picker
                        class="ma-2"
                        v-model="editedItem.end"
                        type="datetime"
                        format="YYYY-MM-DD HH:mm"
                        width="400"
                        placeholder="Hasta"
                      ></date-picker>
                      <!-- End Date picker -->
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" text @click="close">
                    Cancelar
                  </v-btn>
                  <v-btn color="primary" text @click="save">
                    Guardar
                  </v-btn>
                </v-card-actions>
              </v-card>
              <!-- Edit reservas -->
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card flex>
                <v-card-title class="headline justify-center"
                  >¿Desea elminar la reserva?</v-card-title
                >
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" text @click="closeDelete"
                    >Cancelar</v-btn
                  >
                  <v-btn color="primary" text @click="deleteItemConfirm"
                    >Aceptar</v-btn
                  >
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
      </v-data-table>
      <!-- End Table -->
    </v-container>
  </div>
</template>

<script>
import DatePicker from "vue2-datepicker";
import "vue2-datepicker/index.css";
import Vue from "vue";
import moment from "../plugins/moment";
import vis from "../plugins/vis";

Vue.use(vis);

export default {
  components: { DatePicker },

  data() {
    return {
      // vis
      container: "",
      items: [],
      options: {},
      timeline: "",
      arreglo: [
        {
          id: 1,
          content: "Julio Gomez",
          start: "2019-03-1 10:00",
          end: "2019-03-1 10:30",
        },
        {
          id: 2,
          content: "Reserva 3",
          start: "2019-03-1 11:00",
          end: "2019-03-1 11:45",
        },
      ],
      // table
      dialog: false,
      dialogDelete: false,

      headers: [
        {
          text: "Nombre",
          align: "start",
          sortable: true,
          value: "content",
        },
        { text: "Horario inicio", value: "start" },
        { text: "Horario fin", value: "end" },
        {
          text: "Acciones",
          value: "actions",
          sortable: false,
          align: "center",
        },
      ],
      editedIndex: -1,
      editedItem: {
        content: "",
        start: "",
        end: "",
      },
      defaultItem: {
        content: "",
        start: "",
        end: "",
      },
    };
  },
  mounted() {
    // LocalStorage
    this.checkArreglo();

    // Vis
    this.container = document.getElementById("visualization");
    this.items = new vis.DataSet(this.arreglo);
    this.options = {
      //stack: false,
      orientation: {
        axis: "top",
        item: "top",
      },
      //zoomMax: 31536000000, // just one year
      zoomMax: 87600900, // 10,000 years is maximum possible
      zoomMin: 10000000, // 10ms
    };
    this.timeline = new vis.Timeline(
      this.container,
      this.arreglo,
      this.options
    );
    /**
     * Move the timeline a given percentage to left or right
     * @param {Number} percentage   For example 0.1 (left) or -0.1 (right)
     */

    // Using slider to zoomIn or zoomOut
    document
      .getElementById("sliderZoom")
      .addEventListener("input", function () {
        var value = this.value;
        if (value < 0) {
          let start = moment().year(moment().year() - 100000), // to adjust with options
            end = moment().year(moment().year() + 1);
          this.timeline.zoomOut(-value);
          if (value === "-1") this.timeline.setWindow(start, end);
        } else if (value > 0) {
          let start = moment(),
            end = moment(moment().utc() + 10);
          this.timeline.zoomIn(value);
          if (value === "1") this.timeline.setWindow(start, end);
        } else {
          this.timeline.fit(this.items.getIds());
          this.value = 0;
        }
      });
  },
  methods: {
    resetTimeline() {
      let items = new vis.DataSet(this.arreglo);
      this.timeline.destroy();
      this.timeline = new vis.Timeline(this.container, items, this.options);
    },

    resetZoom() {
      // To reset zoom initial state
      document.getElementById("fit").onclick = function () {
        //$('.range').next().text('0'); // set default if to use output with input range
        document.getElementById("sliderZoom").value = 0;
        this.timeline.fit(this.items.getIds());
      };
    },
    move(percentage) {
      let range = this.timeline.getWindow();
      let interval = range.end - range.start;
      this.timeline.setWindow({
        start: range.start.valueOf() - interval * percentage,
        end: range.end.valueOf() - interval * percentage,
      });
    },

    moveLeft() {
      return this.move(0.3);
    },
    moveRight() {
      return this.move(-0.3);
    },

    // table functions
    editItem(item) {
      this.editedIndex = this.arreglo.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.arreglo.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.arreglo.splice(this.editedIndex, 1);
      this.closeDelete();

      this.setArreglo();
      this.resetTimeline();
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
        Object.assign(this.arreglo[this.editedIndex], this.editedItem);
      } else {
        this.arreglo.push(this.editedItem);
      }
      this.close();

      this.setArreglo();
      this.resetTimeline();
    },

    //LocalStorage
    setArreglo() {
      localStorage.setItem("reservas", JSON.stringify(this.arreglo));
    },

    checkArreglo() {
      let reservas = JSON.parse(localStorage.getItem("reservas"));
      if (reservas !== this.arreglo) {
        this.arreglo = reservas;
      }
    },
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Nueva reserva" : "Editar reserva";
    },
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },
};
</script>

<style scoped>
@import "../assets/style.css";
@import "../assets/full-style.css";

.arrow-left {
  position: absolute;
  top: 50%;
  left: 20px;
  z-index: 999;
}
.arrow-right {
  position: absolute;
  top: 50%;
  right: 20px;
  z-index: 999;
}
</style>