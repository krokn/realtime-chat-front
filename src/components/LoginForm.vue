<template>
  <div class="login-form">
    <h2>{{ isLogin ? 'Login' : 'Register' }}</h2>
    <form @submit.prevent="submitForm">
      <div class="input-group">
        <label for="username">Username:</label>
        <input type="text" v-model="username" required />
      </div>
      <div class="input-group">
        <label for="password">Password:</label>
        <input type="password" v-model="password" required />
      </div>
      <!-- Показываем поле TelegramID только при регистрации -->
      <div class="input-group" v-if="!isLogin">
        <label for="telegram_id">TelegramID:</label>
        <input type="text" v-model="telegram_id" required />
      </div>
      <button type="submit" class="submit-btn">{{ isLogin ? 'Login' : 'Register' }}</button>
    </form>
    <button @click="toggleForm" class="toggle-btn">{{ isLogin ? 'Register' : 'Login' }}</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      username: '',
      password: '',
      telegram_id: '',
      isLogin: true,
    };
  },
  methods: {
    async submitForm() {
      try {
        const endpoint = this.isLogin ? '/auth/login' : '/auth/register';
        const payload = {
          username: this.username,
          password: this.password,
        };

        // Добавляем TelegramID только при регистрации
        if (!this.isLogin) {
          payload.telegram_id = parseInt(this.telegram_id, 10);
        }

        const response = await axios.post(`http://localhost:8000/api${endpoint}`, payload);

        if (this.isLogin) {
          const token = response.data.token;
          if (token) {
            this.$cookies.set('token', token, '1h');
            this.$emit('loggedIn');
          } else {
            alert('Token not received. Please try again.');
          }
        } else {
          alert('User registered successfully');
          this.isLogin = true;
        }
      } catch (error) {
        console.error(error);
        const errorMessage = error.response?.data?.detail || 'Something went wrong';
        alert(`Error: ${errorMessage}`);
      }
    },
    toggleForm() {
      this.isLogin = !this.isLogin;
    },
  },
  mounted() {
    this.$cookies.config('1h');
  },
};
</script>

<style scoped>
.login-form {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #1a1a1a;
  color: #e0e0e0;
  padding: 20px;
}

h2 {
  color: #a020f0;
  margin-bottom: 20px;
  font-size: 28px;
  text-align: center;
}

.input-group {
  margin-bottom: 20px;
  width: 100%;
  max-width: 320px; /* Ограничим ширину формы */
}

label {
  color: #e0e0e0;
  font-size: 18px;
}

input[type="text"], input[type="password"] {
  width: 100%;
  padding: 12px;
  margin-top: 5px;
  border: 1px solid #a020f0;
  border-radius: 5px;
  background-color: #2b2b2b;
  color: #e0e0e0;
  outline: none;
  box-sizing: border-box;
}

input[type="text"]:focus, input[type="password"]:focus {
  border-color: #c71585;
}

.submit-btn, .toggle-btn {
  width: 100%;
  max-width: 320px; /* Устанавливаем одинаковую ширину для кнопок */
  padding: 12px;
  margin-top: 10px;
  border: 2px solid white;
  border-radius: 5px;
  background-color: #a020f0;
  color: white;
  cursor: pointer;
  font-size: 18px;
  text-align: center;
  box-sizing: border-box;
}

.submit-btn:hover, .toggle-btn:hover {
  background-color: #c71585;
}

/* Отступы между элементами формы */
.input-group + .input-group {
  margin-top: 15px;
}

.submit-btn + .toggle-btn {
  margin-top: 15px;
}

@media (max-width: 600px) {
  .login-form {
    padding: 10px;
  }

  h2 {
    font-size: 24px;
  }

  .input-group {
    margin-bottom: 15px;
  }

  .submit-btn, .toggle-btn {
    font-size: 16px;
  }
}

</style>
