<template>
  <v-main class="list">
    <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

    <v-card>
      <v-card-title>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          single-line
          hide-details
        >
        </v-text-field>
        <v-spacer></v-spacer>
        <v-btn color="success" dark @click="dialog = true"> Tambah </v-btn>
      </v-card-title>
      <v-data-table
        v-model="select"
        :headers="headers"
        :items="todos"
        :search="search"
        show-select
        class="elevation-1"
        :expanded.sync="expanded"
        item-key="note"
        show-expand
      >
        <template v-slot:[`item.actions`]="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>

        <template v-slot:[`item.priority`]="{ item }">
          <v-chip
            v-show="item.priority == 'Penting'"
            color="red"
            label
            outlined
            >{{ item.priority }}</v-chip
          >
          <v-chip
            v-show="item.priority == 'Tidak Penting'"
            color="green"
            label
            outlined
            >{{ item.priority }}</v-chip
          >
          <v-chip
            v-show="item.priority == 'Biasa'"
            color="blue"
            label
            outlined
            >{{ item.priority }}</v-chip
          >
        </template>
        <template v-slot:expanded-item="{ headers, item }">
          <td :colspan="headers.length">
            <h1><strong>Note: </strong></h1>
            {{ item.note }}
          </td>
        </template>
        <template v-slot:[`item.selects`]="{ item }">
          <v-simple-checkbox
            multiple
            :key="item"
            @click.capture.stop="checkItem(item)"
          />
        </template>
      </v-data-table>
    </v-card>

    <v-dialog v-model="dialogDelete" max-width="500px">
      <v-card>
        <v-card-title class="headline"
          >Are you sure you want to delete this item?</v-card-title
        >
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
          <v-btn color="blue darken-1" text @click="deleteItemConfirm"
            >OK</v-btn
          >
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Form Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field v-model="formTodo.task" label="Task" required>
            </v-text-field>
            <v-select
              v-model="formTodo.priority"
              :items="['Penting', 'Biasa', 'Tidak penting']"
              label="Priority"
              required
            >
            </v-select>
            <v-textarea v-model="formTodo.note" label="Note" required>
            </v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancel"> Cancel </v-btn>
          <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-card class="mt-5">
      <v-card-title class="float-left">Delete Multipe</v-card-title>

      <br />
      <br />
      <br />
      <v-list-item v-for="(item, i) in select" :key="i">
        <v-list-item-content>
          <v-list-item-title>• {{ item.task }}</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <v-btn color="error ml-4 mt-5 mr-5 mb-3" dark @click="deleteItemMultiple">
        Hapus Semua
      </v-btn>
    </v-card>
  </v-main>
</template>
<script>
export default {
  name: "ListTugas",
  data() {
    return {
      select: [],
      search: null,
      dialog: false,
      dialogDelete: false,

      headers: [
        {
          text: "Task",
          align: "start",
          sortable: true,
          value: "task",
        },
        { text: "Priority", value: "priority" },
        { text: "Actions", value: "actions" },
      ],
      todos: [
        {
          task: "bernafas",
          priority: "Penting",
          note: "huffttt",
        },
        {
          task: "nongkrong",
          priority: "Tidak Penting",
          note: "bersama tman2",
        },
        {
          task: "masak",
          priority: "Biasa",
          note: "masak air 500ml",
        },
      ],
      editTodo: {
        task: "",
        priority: "",
        note: "",
      },
      defaultItem: {
        task: "",
        priority: "",
        note: "",
      },
      edit: -1,
      delete: -1,
      formTodo: {
        task: null,
        priority: null,
        note: null,
      },
    };
  },

  methods: {
    save() {
      if (this.edit > -1) {
        (this.editTodo.task = this.formTodo.task),
          (this.editTodo.priority = this.formTodo.priority),
          (this.editTodo.note = this.formTodo.note),
          (this.dialog = false);
      } else {
        this.todos.push(this.formTodo);
        this.resetForm();
        this.dialog = false;
      }
    },
    cancel() {
      this.resetForm();
      this.dialog = false;
    },
    resetForm() {
      this.formTodo = {
        task: null,
        priority: null,
        note: null,
      };
    },
    editItem(item) {
      (this.edit = this.todos.indexOf(item)),
        (this.formTodo.task = item.task),
        (this.formTodo.priority = item.priority),
        (this.formTodo.note = item.note),
        (this.dialog = true);
      this.editTodo = item;
    },
    deleteItem(item) {
      this.delete = this.todos.indexOf(item);
      (this.editTodo = item), (this.dialogDelete = true);
    },

    deleteItemConfirm() {
      this.todos.splice(this.delete, 1);
      this.closeDelete();
    },
    close() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editTodo = Object.assign({}, this.defaultItem);
        this.delete = -1;
      });
    },
    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editTodo = Object.assign({}, this.defaultItem);
        this.delete = -1;
      });
    },
    checkItem(item) {
      if (this.select.includes(item)) {
        this.select.splice(this.select.indexOf(item), 1);
      } else {
        this.select.push(item);
      }
    },
    deleteItemMultiple() {
      for (var i = 0; i < this.select.length; i++) {
        const index = this.todos.indexOf(this.select[i]);
        this.todos.splice(index, 1);
      }
    },
  },
};
</script>