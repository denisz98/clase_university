<!-- eslint-disable-next-line vue/multi-word-component-names -->
<template>
<v-container mt-5>
  <v-data-table
    :headers="headers"
    :items="estudiante"
    :sort-by="search"
    class="elevation-1" dark
  >
    <template #top>
      <v-toolbar flat>
        <v-toolbar-title>Estudiantes</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>

        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template #activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              Add New
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.nombre"
                      label="Nombre"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.CI"
                      label="CI"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.direccion"
                      label="Dirección"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.telefono"
                      label="Teléfono"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.anno_academico"
                      label="Año académico"
                    ></v-text-field>
                  </v-col>

                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancel</v-btn
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
    <template  #item.actions="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template #no-data>
      <v-btn color="primary" @click="initialize"> Reset </v-btn>
    </template>
  </v-data-table>
  </v-container>
</template>
<script>
const axios = require('axios')
export default {
  data: () => ({
    search: '',
    dialog: false,
    dialogDelete: false,
    // eslint-disable-next-line no-sparse-arrays
    headers: [
      {
        text: 'Nombres',
        align: 'start',
        sortable: false,
        value: 'nombre',
      },
      { text: 'CI', value: 'CI' },
      { text: 'Dirección', value: 'direccion' },
      { text: 'Teléfono', value: 'telefono' },
      { text: 'Año académico', value: 'anno_academico' },
      { text: 'Actions', value: 'actions', sortable: false },

      ,
    ],
    estudiante: [],
    editedIndex: -1,
    editedItem: {
      nombre: '',
      CI: '',
      direccion: '',
      telefono: '',
      anno_academico: '',
    },
    defaultItem: {
      nombre: '',
      CI: '',
      direccion: '',
      telefono: '',
      anno_academico: '',
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },

  created() {
    this.initialize()
  },

  methods: {
    async initialize() {
      // eslint-disable-next-line no-unused-vars
      await axios
        .get('http://localhost:1337/api/estudiantes')
        .then((response) => {
          response.data.data.forEach((item) => {
            console.log(item.id)
            this.estudiante.push({
              id: item.id,
              nombre: item.attributes.nombre,
              CI: item.attributes.CI,
              direccion: item.attributes.direccion,
              telefono: item.attributes.telefono,
              anno_academico: item.attributes.anno_academico,
            })
          })
        })
        .catch((error) => {
          console.error(error)
        })
    },

    editItem(item) {
      this.editedIndex = this.estudiante.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

   deleteItem(item) {
     this.editedIndex = this.estudiante.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      await axios
      .delete('http://localhost:1337/api/estudiantes/'+this.estudiante[this.editedIndex].id)
      .then((response) => console.log(response))
      .catch((error) => console.log("Error: "+ error));
      this.estudiante = [];
      await this.initialize();
      this.closeDelete();
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    async save() {
      try {
        await this.$strapi.create('estudiante', {
          data: this.editedItem,
        })
      } catch (error) {
        console.log(error)
      }S
      this.close()
    },
  },
}
</script>


