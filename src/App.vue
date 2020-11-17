<template>
  <v-app>
    <div class="container" style="max-width: -webkit-fill-available">
      <v-toolbar
        dark
        dense
        src="https://cdn.vuetifyjs.com/images/backgrounds/vbanner.jpg"
      >
        <v-toolbar-title>Users CRUD</v-toolbar-title>
      </v-toolbar>

      <v-data-table
        :headers="headers"
        :items="users"
        sort-by="calories"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>My Users</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  dark
                  class="mb-2"
                  v-bind="attrs"
                  v-on="on"
                >
                  New User
                </v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-form     
                    ref="form"
                    v-model="valid"
                    lazy-validation>
                    <v-row>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.name.firstName"
                          label="First Name"
                          :rules="basicRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.name.lastName"
                          label="Last Name"
                          :rules="basicRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="4">
                        <v-text-field
                          v-model="editedItem.username"
                          label="Username"
                          :rules="usernameRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="6">
                        <v-text-field
                          v-model="editedItem.identification.type"
                          label="ID Type"
                          :rules="basicRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="6">
                        <v-text-field
                          v-model="editedItem.identification.number"
                          label="ID Number"
                          type="number"
                          :rules="basicRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="12">
                        <v-text-field
                          v-model="editedItem.password"
                          label="Password"
                          type="password"
                          :rules="basicRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="12">
                        <v-text-field
                          v-model="editedItem.photoURL"
                          label="Photo URL"
                          :rules="basicRules"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" sm="6" md="6">
                        <v-checkbox
                          v-model="editedItem.active"
                          label="Active?"
                        ></v-checkbox>
                      </v-col>
                    </v-row>
                  </v-form>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" text @click="close">
                    Cancel
                  </v-btn>
                  <v-btn :disabled="!valid" color="blue darken-1" text @click="save"> Save </v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <v-dialog v-model="dialogDelete" max-width="500px">
              <v-card>
                <v-card-title class="headline"
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
        <template v-slot:item.actions="{ item }">
          <v-icon small class="mr-2" @click="editItem(item)">
            mdi-pencil
          </v-icon>
          <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize"> Reset </v-btn>
        </template>
      </v-data-table>
    </div>
  </v-app>
</template>
<script>
import axios from "axios";

export default {
  data: () => ({
    dialog: false,
    dialogDelete: false,
    valid: true,
    newUser: "",
    users: [],
    editedIndex: -1,
     basicRules: [
        v => !!v || 'This Field is required',
      
      ],
       usernameRules: [
        v => !!v || 'This Field is required',
        v => (v && v.length >= 8) || 'Username must have at least 8 characters',
      ],
    editedItem: {
      name: {
        firstName: "",
        lastName: "",
      },
      username: "",
      identification: {
        type: "",
        number: "",
       
      },
      password: "",
      photoURL: "",
      active: true,
    },
    defaultItem: {
      name: {
        firstName: "",
        lastName: "",
      },
      username: "",
      identification: {
        number: "",
        type: "",
      },
      password: "",
      photoURL: "",
      active: true,
    },
    headers: [
      { text: "First Name", value: "name.firstName" },
      { text: "Last Name", value: "name.lastName" },
      { text: "Username", value: "username" },
      { text: "ID Type", value: "identification.type" },
      { text: "ID Number", value: "identification.number" },
      { text: "Password", value: "password", sortable: false },
      { text: "Photo URL", value: "photoURL", sortable: false },
      { text: "Active", value: "active" },
      { text: "Actions", value: "actions", sortable: false },
    ],
  }),
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New User" : "Edit User";
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
    this.initialize();
  },
  methods: {
    initialize() {
      axios
        .get("https://taller-nodeback.herokuapp.com/users")
        .then((response) => {
          this.users = response.data;
        })
        .catch((error) => console.log(error));
    },

    editItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      
      axios.delete("https://taller-nodeback.herokuapp.com/user/" + this.users[this.editedIndex]._id).then((response) => {
        console.log(response.data);
      });
      this.users.splice(this.editedIndex, 1);
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
        console.log(this.users[this.editedIndex]);
        Object.assign(this.users[this.editedIndex], this.editedItem);
        axios
          .put(
            "https://taller-nodeback.herokuapp.com/users/" + this.users[this.editedIndex]._id, this.users[this.editedIndex]
          )
          .then((response) => {
            console.log(response.data);
          });
      } else {
        axios
          .post("https://taller-nodeback.herokuapp.com/user/add", {
            ...this.editedItem,
          })
          .then((response) => {
            console.log(response.data);
          });
        this.users.push(this.editedItem);
      }

      this.close();
    },
  },
};
</script>