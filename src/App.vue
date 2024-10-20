<template>
  <div id="app">
    <!-- Отображаем компонент Login, если пользователь не вошел в систему -->
    <LoginForm v-if="!isLoggedIn" @loggedIn="handleLogin" />
    <!-- Отображаем компонент Chat, если пользователь вошел в систему -->
    <ChatForm v-if="isLoggedIn" @logout="handleLogout" />
  </div>
</template>

<script>
import LoginForm from './components/LoginForm.vue'; // Используем правильный путь к компоненту
import ChatForm from './components/ChatForm.vue';   // Используем правильный путь к компоненту

export default {
  name: 'App',
  data() {
    return {
      isLoggedIn: false, // Показываем форму логина по умолчанию
    };
  },
  methods: {
    handleLogin() {
      this.isLoggedIn = true; // Переключаем на компонент чата после успешного логина
    },
    handleLogout() {
      this.isLoggedIn = false; // Возвращаем к форме логина при выходе из аккаунта
    },
    checkAuth() {
      const token = this.$cookies.get('token');
      if (token) {
        this.isLoggedIn = true; // Если есть токен, то пользователь считается аутентифицированным
      }
    },
  },
  mounted() {
    this.checkAuth(); // Проверяем токен при загрузке страницы
  },
  components: {
    LoginForm,
    ChatForm,
  },
};
</script>

<style>
/* Добавьте любые стили, если нужно */
</style>
