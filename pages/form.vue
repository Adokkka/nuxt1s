<template>
  <div id="app">
    <h2>Форма данных</h2>
    <form @submit.prevent="saveData">
      <div>
        <label for="firstName">Имя:</label>
        <input id="firstName" v-model="form.firstName" type="text" />
      </div>
      <div>
        <label for="lastName">Фамилия:</label>
        <input id="lastName" v-model="form.lastName" type="text" />
      </div>
      <div>
        <label for="age">Возраст:</label>
        <input id="age" v-model="form.age" type="number" />
      </div>
      <button type="submit">Сохранить</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        firstName: "",
        lastName: "",
        age: "",
      },
      isSaved: true,
    };
  },
  watch: {
    form: {
      handler() {
        this.isSaved = false;
      },
      deep: true,
    },
  },
  methods: {
    saveData() {
      this.isSaved = true;
      alert("Данные сохранены!");
    },
  },
  beforeRouteLeave(to, from, next) {
    if (!this.isSaved) {
      const answer = confirm(
        "У вас есть несохраненные изменения. Хотите сохранить перед уходом?"
      );
      if (answer) {
        this.saveData();
      }
    }
    next();
  },
};
</script>

<style scoped>
form div {
  margin-bottom: 10px;
}
</style>
