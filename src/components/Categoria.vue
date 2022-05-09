<template>
  <v-container>
    <v-data-table
      :headers="headers"
      :items="categorias"
      :search="search"
      sort-by="calories"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Categorias</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-text-field
            label="Buscar"
            class="text-xs-center"
            v-model="search"
            append-icon="search"
            single-line
          ></v-text-field>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="400px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Nuevo
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="12" md="12">
                      <v-text-field
                        v-model="nombre"
                        label="Nombre"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="12" md="12">
                      <v-text-field
                        v-model="descripcion"
                        label="Descripcion"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="12" md="12" v-show="valida">
                      <div
                        class="red--text"
                        v-for="v in validaMensaje"
                        :key="v"
                        v-text="v"
                      ></div>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancelar
                </v-btn>
                <v-btn color="blue darken-1" text @click="guardar">
                  Guardar
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5"
                >¿Estas seguro de eliminar la categoría? {{_id}}</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete()"
                  >Cancelar</v-btn
                >
                <v-btn color="blue darken-1" text @click="eliminar()"
                  >Aceptar</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="adModal" max-width="290">
            <v-card>
              <v-card-title class="headline" v-if="adAccion == 1">
                Activar Categoría
              </v-card-title>
              <v-card-title class="headline" v-if="adAccion == 2">
                Desactivar Categoría
              </v-card-title>

              <v-card-text>
                ¿Estas seguro de <span v-if="adAccion==1">activar</span> 
                <span v-if="adAccion==2">desactivar</span> la categoría {{adNombre}}
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red darken-1" text @click="activarDesactivarCerrar()"
                  >Cancelar</v-btn
                >
                <v-btn color="blue darken-1" text @click="activar()" v-if="adAccion == 1"
                  >Activar</v-btn
                >
                <v-btn color="blue darken-1" text @click="desactivar()" v-if="adAccion == 2"
                  >Desactivar</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>

      <!--Colocar el estado-->
      <template v-slot:[`item.estado`]="{ item }">
        <div v-if="item.estado">
          <span class="blue--text">Activo</span>
        </div>
        <div v-else>
          <span class="red--text">Inactivo</span>
        </div>
      </template>

      <template v-slot:[`item.actions`]="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)"> edit </v-icon>
        <v-icon small class="mr-2" @click="deleteItem(item)"> delete</v-icon>
        <template v-if="item.estado">
          <v-icon small @click="activarDesactivarMostrar(2, item)">
            block
          </v-icon>
        </template>
        <template v-else>
          <v-icon small @click="activarDesactivarMostrar(1, item)">
            check
          </v-icon>
        </template>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="listar()"> Resetear </v-btn>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data: () => ({
    dialog: false,
    search: "",
    categorias: [],
    dialogDelete: false,
    headers: [
      { text: "Nombre", value: "nombre", sortable: true },
      { text: "Descripcion", value: "descripcion", sortable: false },
      { text: "Estado", value: "estado", sortable: false },
      { text: "Acciones", value: "actions", sortable: false },
    ],
    editedIndex: -1,
    _id: null,
    nombre: "",
    descripcion: "",
    valida: 0,
    validaMensaje: [],
    adModal: 0,
    adAccion: 0,
    adNombre: "",
    adId: "",
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Nuevo registro" : "Editar registro";
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

  created() {
    this.listar();
  },

  methods: {
    listar() {
      let me = this;
      axios.get('categoria/list')
        .then((response) => {
          me.categorias = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    validar() {
      this.valida = 0;
      this.validaMensaje = [];
      if (this.nombre.length < 1 || this.nombre.length > 50) {
        this.validaMensaje.push(
          "El nombre de la categoría debe tener entre 1-50 caracteres."
        );
      }
      if (this.descripcion.length > 256) {
        this.validaMensaje.push(
          "La descripcion de la categoría no debe tener más de 256 caracteres."
        );
      }
      if (this.validaMensaje.length) {
        this.valida = 1;
      }

      return this.valida;
    },

    limpiar() {
      this._id = null;
      this.nombre = "";
      this.descripcion = "";
      this.valida = 0;
      this.validaMensaje = [];
      this.editedIndex = -1;
    },

    guardar() {
      let me = this;
      if (this.validar()) {
        return;
      }
      if (this.editedIndex > -1) {
        //Código para editar
        axios.put("categoria/update", {
            '_id': this._id,
            'nombre': this.nombre,
            'descripcion': this.descripcion,
          })
          .then((response) => {
            me.limpiar();
            me.close();
            me.listar();
          })
          .catch((error) => {
            console.log(error);
          });
      } else {
        //codigo para guardar
        axios
          .post("categoria/add", {
            'nombre': this.nombre,
            'descripcion': this.descripcion,
          })
          .then((response) => {
            me.limpiar();
            me.close();
            me.listar();
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },
    editItem(item) {
      this._id = item._id;
      this.nombre = item.nombre;
      this.descripcion = item.descripcion;
      this.dialog = true;
      this.editedIndex = 1;
    },
   
    activar(){
      let me = this;
      axios.put("categoria/activate", {'_id': this.adId})
          .then((response) => {
            this.adModal = 0;
            this.adAccion = 0;
            this.adNombre = '';
            this.adId = '';
            this.listar();
          })
          .catch((error) => {
            console.log(error);
          });

    },

    desactivar(){
      let me = this;
      axios.put("categoria/deactivate", {'_id': this.adId})
          .then((response) => {
            this.adModal = 0;
            this.adAccion = 0;
            this.adNombre = '';
            this.adId = '';
            me.listar();
          })
          .catch((error) => {
            console.log(error);
          });

    },

    deleteItem(item) {
      this._id = item._id;
      this.dialogDelete = true;
    },

    eliminar(){
      let me = this;
      axios.delete(`categoria/remove/${this._id}`)
          .then((response) => {
            this.adModal = 0;
            this.adAccion = 0;
            this.adNombre = '';
            this.adId = '';
            me.listar();
            me.closeDelete();
          })
          .catch((error) => {
            console.log(error);
          });
    },

    deleteItemConfirm() {
      let me = this;
      axios.delete('categoria/remove',{'_id' : this._id })
        .then((response) => {
          me.limpiar();
          me.close();
          me.listar();
          me.closeDelete();
        })
        .catch((error) => {
          console.log(error);
        });
    },

     activarDesactivarMostrar(accion, item) {
      this.adModal = 1;
      this.adNombre = item.nombre;
      this.adId = item._id;

      if (accion == 1) {
        this.adAccion = 1;
      } else if (accion == 2) {
        this.adAccion = 2;
      } else {
        this.adModal = 0;
      }
    },

    close() {
      this.dialog = false;
      this.valida = 0;
      this.validaMensaje = [];
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },
    activarDesactivarCerrar(){
      this.adModal = 0;
    },
  },
};
</script>