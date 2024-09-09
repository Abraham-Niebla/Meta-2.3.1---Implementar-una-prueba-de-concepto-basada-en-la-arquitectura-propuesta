<template>
<v-container fluid class="fill-space no-gutters">
<v-row class="fill-space no-gutters">
<v-col cols="12" class="fill-space no-gutters">
  <v-data-table
    v-model:expanded="expanded"
    :headers="headers"
    :items="alertas"
    class="elevation-1 fill-height"
    item-value="alertaName"
    show-expand
    cols="12"
    :height="tableHeight">

    <!--Barra superior-->
    <template v-slot:top>
      <!--Dentro de la barra superior-->
      <v-toolbar flat color="indigo-lighten-5">
        <v-toolbar-title class="text-indigo-darken-3" cols="12">
          <h4>Alertas</h4></v-toolbar-title>

        <v-spacer />
        <v-divider
          :thickness="3"
          color="teal-darken-3"
          class="mx-4"
          inset
          vertical/>

        <!--Boton Reinicio-->
        <v-btn color="red-darken-4" @click="deleteAll" density="compact" icon="mdi-delete" />


        <!--Confirmacion de eliminacion de elementos-->
        <v-dialog
          v-model="dialogDeleteAll"
          width="400px"
          class="text-center">
          <v-card>
            <v-card-title class="text-h5">
              ¿Eliminar todas las alertas?
            </v-card-title>
            <v-card-actions>
              <v-spacer />
              <v-btn
                color="green-darken-4"
                variant="text"
                @click="closeDeleteAll">
                <v-icon class="me-2"> mdi-cancel </v-icon>
                Cancelar
              </v-btn>
              <v-btn
                color="red-darken-4"
                variant="text"
                @click="deleteAllConfirm">
                <v-icon class="me-2"> mdi-delete </v-icon>
                Eliminar
              </v-btn>
              <v-spacer />
            </v-card-actions>
          </v-card>
        </v-dialog>


        <v-divider
          :thickness="3"
          color="teal-darken-3"
          class="mx-4"
          inset
          vertical/>

        <!--Boton Nuevo Elemento-->
        <v-dialog v-model="dialog" width="1200px">
          <template v-slot:activator="{ props }">
            <v-btn color="light-green-darken-4" dark v-bind="props" density="compact" icon="mdi-alarm-plus" />
          </template>

          <!--Edicion de elemento existente o nuevo-->
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formtitleElement }}</span>
            </v-card-title>

            <!--Editor de elemento-->
            <v-card-text>
              <v-form
                v-model="form"
                @submit.prevent="save">
              <v-container>
                <v-row>
                  <v-col cols="12" class="text-indigo-darken-4">
                    <v-text-field :rules="[required]"
                      v-model="editedItem.alertaName"
                      label="Alerta"/>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" class="text-black">
                    <v-textarea  v-model="editedItem.desc" label="Descripción" />
                  </v-col>
                </v-row>

                <v-row>
                  <v-col cols="4">
                    <v-text-field :rules="[required]"
                      v-model="editedItem.time"
                      label="Hora"
                      type="time" />
                  </v-col>
                  <v-col cols="4">
                    <v-text-field :rules="[required]"
                      v-model="editedItem.date"
                      label="Fecha"
                      type="date" />
                  </v-col>
                  <v-col cols="4" class="text-light-blue-darken-4">
                    <v-select :rules="[required]"
                        v-model="editedItem.med"
                        label="Medio"
                        :items="mediosList"
                        item-title="medio"
                        item-children="icon"
                        chips
                        size="x-large" />
                  </v-col>
                </v-row>


              </v-container>
              </v-form>
            </v-card-text>

            <!--Confirmacion de editor de elemento-->
            <v-card-actions>
              <v-spacer />
              <v-btn color="red-darken-4" variant="text" @click="close">
                <v-icon class="me-2"> mdi-cancel </v-icon>
                Cancelar
              </v-btn>
              <v-btn color="purple-darken-1" variant="text" @click="save" :disabled="!form">
                <v-icon class="me-2"> mdi-content-save </v-icon>
                Guardar
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-divider
          :thickness="3"
          color="teal-darken-3"
          class="mx-4"
          inset
          vertical/>

        <!--Confirmacion de eliminacion de elemento-->
        <v-dialog
          v-model="dialogDeleteItem"
          width="400px"
          class="text-center">
          <v-card>
            <v-card-title class="text-h5"
              >¿Eliminar esta alerta?</v-card-title>
            <v-card-actions>
              <v-spacer />
              <v-btn
                color="green-darken-4"
                variant="text"
                @click="closeDeleteItem">
                <v-icon class="me-2"> mdi-cancel </v-icon>
                Cancelar
              </v-btn>
              <v-btn
                color="red-darken-4"
                variant="text"
                @click="deleteItemConfirm">
                <v-icon class="me-2"> mdi-delete </v-icon>
                Eliminar
              </v-btn>
              <v-spacer />
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>

    <!-- Botón para más información -->
    <template v-slot:expanded-row="{ columns, item }">
      <tr>
        <td :colspan="columns.length">
          {{ item.desc }}
        </td>
      </tr>
    </template>

    <!--Columna de Acciones-->
    <template v-slot:item.actions="{ item }">
      <v-icon
        size="small"
        class="me-2"
        color="cyan-darken-2"
        @click="editItem(item)">
        mdi-pencil
      </v-icon>
      <v-icon size="small" color="red-darken-2" @click="deleteItem(item)">
        mdi-delete
      </v-icon>
    </template>

    <!--Sin datos en la tabla-->
    <template v-slot:no-data>
      <h2 style="color: #00796B9c">¡Presione sobre el simbolo</h2>
      <v-icon size="x-large" style="color: #00796B9c">mdi-alarm-plus</v-icon>
      <h2 style="color: #00796B9c"> para agregar una nueva alerta!</h2>
    </template>
  </v-data-table>
</v-col>
</v-row>
</v-container>
</template>

<script setup>
import { ref, computed, onMounted  } from "vue";

const expanded = ref([]);

const headers = [
  { title: "Alerta", key: "alertaName", align: "justify", width: '40%' },
  { title: "Hora", key: "time", align: "center", width: '15%' },
  { title: "Fecha", key: "date", align: "center", width: '15%' },
  { title: "Medio", key: "med", align: "center", width: '15%' },
  { title: "", key: "data-table-expand", align: "center", width: '5%'},
  { title: "", key: "actions", sortable: false, align: "center", width: '10%' },
];

const mediosList = [
  {id: 1, medio: "WhatsApp"},
  {id: 2, medio: "Messenger"},
  {id: 3, medio: "Telegram"},
  {id: 4, medio: "Signal"},
  {id: 5, medio: "e-mail"},
  {id: 6, medio: "SMS"},
]

const defaultItem = {
  alertaName: "",
  desc: "",
  time: "",
  date: "",
  med: "",
};

const form = ref(false);

const tableHeight = ref('100%'); // Asigna altura inicial de la tabla al 100%
let width; //Ancho de la pantalla
let heigh; //Alto de la pantalla
const dialog = ref(false);
const dialogDeleteItem = ref(false);
const dialogDeleteAll = ref(false);
const alertas = ref([]);
const editedIndex = ref(-1);
const editedItem = ref({ ...defaultItem }); //Copia del modelo de item

//Crea el título del cuadro de diálogo para crear o editar un elemento
const formtitleElement = computed(() => {
  return editedIndex.value === -1 ? "Nueva alerta" : "Editar alerta";
});

//Inicializa la tabla con los datos originales
const initialize = async () => {
  alertas.value = [];
};

//Abre cuadro de dialogo para añadir/editar elemento
const editItem = (item) => {
  editedIndex.value = alertas.value.indexOf(item);
  editedItem.value = { ...item };
  dialog.value = true;
};

//Abre cuadro de diálogo para borrar elemento
const deleteItem = (item) => {
  editedIndex.value = alertas.value.indexOf(item);
  editedItem.value = { ...item };
  dialogDeleteItem.value = true;
};

//Abre cuadro de diálogo para borrar todo
const deleteAll = () => {
  dialogDeleteAll.value = true;
};

//Elimina el elemento seleccionado
const deleteItemConfirm = () => {
  alertas.value.splice(editedIndex.value, 1);
  closeDeleteItem();
};

//Elimina los elementos
const deleteAllConfirm = () => {
  alertas.value = [];
  closeDeleteAll();
};

//Cierra el cuadro de diálogo de añadir/editar elemento
const close = () => {
  dialog.value = false;
  editedItem.value = { ...defaultItem };//Copia del modelo de item
  editedIndex.value = -1;
};

//Cierra el cuadro de diálogo de borrado de elemento
const closeDeleteItem = () => {
  dialogDeleteItem.value = false;
  editedItem.value = { ...defaultItem };
  editedIndex.value = -1;
};

//Cierra el cuadro de diálogo de borrado de elemento
const closeDeleteAll = () => {
  dialogDeleteAll.value = false;
};

//Guarda un elemento editado o nuevo
const save = () => {
  if (!form.value) return

  //Guarda un elemento editado
  if (editedIndex.value > -1)
  {
    Object.assign(alertas.value[editedIndex.value], editedItem.value);
    close();
  }
  //Guarda un elemento nuevo
  else
  {
    alertas.value.push({ ...editedItem.value });
    close();
  }
};

// Validación de existencia de datos
const required = (v) => {
  return !!v || 'El campo es requerido'
}

initialize();

// Ajusta el tamaño de la tabla en base a la ventana
const adjustTableSize = () => {
  const windowHeight = window.innerHeight;
  const windowWidth = window.innerWidth;
  tableHeight.value = `${windowHeight}px`;

  console.log("windowHeight = ", windowHeight);
  console.log("windowWidth = ", windowWidth);
};

// Escucha el cambio de tamaño de la ventana y ajusta la tabla
onMounted(() => {
  adjustTableSize();
  window.addEventListener('resize', adjustTableSize);
});

</script>

<style scoped>
.fill-space {
  height: 100vh; /* Altura completa de la ventana */
  width: 100vw; /* Ancho completo de la ventana */
  overflow: hidden;
}

.no-gutters {
  padding: 0;
  margin: 0;
}

.v-data-table {
  overflow: auto; /* Permite desplazamiento si hay más contenido */
}
</style>
