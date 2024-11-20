<template>
  <v-container>
    <v-form ref="form" v-model="valid">
      <v-autocomplete
        v-model="selectedUser"
        :items="users"
        item-text="first_name"
        label="Выберите имя"
        @change="fetchUserData"
        :rules="[(v) => !!v || 'Обязательное поле']"
      ></v-autocomplete>

      <v-text-field
        v-model="formData.address"
        label="Адрес"
        :rules="[(v) => !!v || 'Обязательное поле']"
        required
      ></v-text-field>

      <v-text-field
        v-model="formData.age"
        label="Возраст"
        :rules="[(v) => !!v || 'Обязательное поле']"
        required
      ></v-text-field>

      <v-radio-group v-model="formData.gender" row>
        <v-radio label="Мужской" value="male"></v-radio>
        <v-radio label="Женский" value="female"></v-radio>
      </v-radio-group>

      <v-text-field
        v-model="formData.favoriteFood"
        label="Любимая еда"
      ></v-text-field>

      <v-btn :disabled="!valid" @click="submitForm" color="primary">
        Отправить заявку
      </v-btn>
    </v-form>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      valid: false,
      selectedUser: null,
      users: [],
      formData: {
        address: "",
        age: "",
        gender: "",
        favoriteFood: "",
      },
    };
  },
  mounted() {
    this.fetchUsers();
  },
  methods: {
    async fetchUsers() {
      try {
        const response = await axios.get("http://intra.isker.kz/api/v1/users");
        this.users = response.data;
      } catch (error) {
        console.error("Ошибка при загрузке пользователей:", error);
      }
    },
    async fetchUserData() {
      if (this.selectedUser) {
        try {
          const userId = this.selectedUser.username;
          const response = await axios.get(
            `nls-1c/HttpService/hs/intra/getUserInfo/${userId}`
          );
          const userData = response.data;
          this.formData = {
            address: userData.address || "",
            age: userData.age || "",
            gender: userData.gender || "",
            favoriteFood: userData.favoriteFood || "",
          };
        } catch (error) {
          console.error("Ошибка при загрузке данных пользователя:", error);
        }
      }
    },
    async submitForm() {
      try {
        const response = await axios.post(
          "http://192.168.0.67/",
          this.formData
        );
        console.log("Заявка успешно отправлена:", response.data);
        alert("Заявка успешно отправлена!");
        this.resetForm();
      } catch (error) {
        console.error("Ошибка при отправке заявки:", error);
        alert("Произошла ошибка при отправке.");
      }
    },
    resetForm() {
      this.$refs.form.reset();
      this.formData = {
        address: "",
        age: "",
        gender: "",
        favoriteFood: "",
      };
    },
  },
};
</script>
