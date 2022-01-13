<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" clipped fixed app>
      <v-list dense>
        <v-list-item
          v-for="file in files"
          :key="file.id"
          :to="`/edit/${file.id}`"
          router
          exact
        >
          <v-list-item-content>
            <v-list-item-title v-text="file.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar clipped-left fixed app>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer app>
      <span>moontai0724 &copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
    <v-tooltip left>
      <template #activator="{ on }">
        <v-btn
          color="green darken-2"
          dark
          fab
          fixed
          right
          bottom
          v-on="on"
          @click="dialog = true"
        >
          <v-icon> mdi-plus </v-icon>
        </v-btn>
      </template>
      <span>新增檔案</span>
    </v-tooltip>
    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card :loading="submitting">
        <v-form v-model="isFormValid" @submit.prevent="submitCreate">
          <v-card-title>
            <span class="text-h5">新增檔案</span>
          </v-card-title>
          <v-card-text>
            <v-text-field
              v-model="newFileName"
              :rules="[(v) => !!v || '請填入檔案名稱']"
              label="新檔案名稱"
              persistent-hint
              required
            ></v-text-field>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              type="button"
              color="red lighten-1"
              text
              @click="closeDialog"
            >
              取消
            </v-btn>
            <v-btn
              type="submit"
              color="green lighten-1"
              text
              :disabled="!isFormValid"
            >
              新增
            </v-btn>
          </v-card-actions>
        </v-form>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import { Manager, Socket } from "socket.io-client";
export default Vue.extend({
  name: "DefaultLayout",
  data() {
    return {
      drawer: false,
      files: [],
      title: "簡易線上共筆",
      dialog: false,
      newFileName: null,
      isFormValid: false,
      submitting: false,
      manager: {} as Manager,
      currentSocket: {} as Socket,
    };
  },
  mounted() {
    this.manager = new Manager(
      (location.protocol === "https:" ? "wss" : "ws") +
        "://" +
        this.$config.WS_HOST,
      {
        reconnectionDelayMax: 10000,
      },
    );
    this.currentSocket = this.manager.socket("/");

    this.currentSocket.emit("FetchAllFileTitles");
    this.currentSocket.on(
      "AllFileTitles",
      (allFiles) => (this.files = allFiles),
    );
  },
  methods: {
    closeDialog() {
      this.newFileName = null;
      this.dialog = false;
      this.submitting = false;
    },
    submitCreate() {
      this.submitting = true;
      this.currentSocket.emit("NewFile", { title: this.newFileName });
      this.currentSocket.on("RequestedFile", (file) => {
        this.$router.push("/edit/" + file.id);
        this.closeDialog();
      });
    },
  },
});
</script>
