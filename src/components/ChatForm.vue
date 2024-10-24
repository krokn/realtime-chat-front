<template>
  <div class="chat-container">
    <div class="chat">
      <h2>Send Message</h2>
      <form @submit.prevent="sendMessage" class="chat-form">
        <div class="input-group">
          <label for="recipient">Recipient Username:</label>
          <input type="text" v-model="recipient" required />
        </div>
        <div class="input-group">
          <label for="message">Message:</label>
          <input type="text" v-model="message" required />
        </div>
        <button type="submit" class="send-btn">Send</button>
      </form>
      <div class="chat-box">
        <div v-for="(msg, index) in chatHistory" :key="index" class="message">
          <p :class="{'message-sent': msg.startsWith('Me'), 'message-received': !msg.startsWith('Me')}">{{ msg }}</p>
        </div>
      </div>
    </div>
    <div class="users-list">
      <h3>Registered Users</h3>
      <ul>
        <li v-for="user in users" :key="user.username">{{ user.username }}</li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      recipient: '',
      message: '',
      socket: null,
      chatHistory: [],
      users: []  // Список пользователей
    };
  },
  methods: {
    sendMessage() {
      if (this.socket && this.socket.readyState === WebSocket.OPEN) {
        const formattedMessage = `${this.recipient}|${this.message}`;
        this.socket.send(formattedMessage);
        this.chatHistory.push(`Me: ${this.message}`);
        this.message = '';
      } else {
        alert('WebSocket connection is not open. Message will not be sent.');
      }
    },
    connectToWebSocket() {
      const token = this.$cookies.get('token');
      this.socket = new WebSocket(`ws://31.129.109.97:80/api/message/ws?token=${token}`);

      this.socket.onmessage = (event) => {
        this.chatHistory.push(event.data);
      };

      this.socket.onclose = () => {
        console.log('WebSocket closed');
      };
    },
    fetchUsers() {
      axios.get('http://31.129.109.97:80/api/user', {
        headers: {
          accept: 'application/json',
        }
      })
      .then(response => {
        this.users = response.data;
      })
      .catch(error => {
        console.error("Error fetching users:", error);
      });
    }
  },
  mounted() {
    this.connectToWebSocket();
    this.fetchUsers();  // Загружаем список пользователей при монтировании компонента
  },
  beforeUnmount() {
    if (this.socket) {
      this.socket.close();
    }
  },
};
</script>

<style scoped>
/* Основной контейнер с более узкими размерами */
.chat-container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  background-color: black; /* Устанавливаем черный фон */
  height: 100vh; /* Высота на весь экран */
  padding: 20px;
  box-sizing: border-box;
}

.chat {
  width: 400px;
  padding: 20px;
  background-color: #1a1a1a; /* Темный фон для чата */
  color: #e0e0e0;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  margin-right: 20px;
}

.users-list {
  width: 200px;
  background-color: #2b2b2b; /* Темный фон для списка пользователей */
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

h2, h3 {
  color: #a020f0;
  margin-bottom: 15px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 10px;
  border-bottom: 1px solid #444;
  color: #e0e0e0;
}

li:last-child {
  border-bottom: none;
}

.chat-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.input-group {
  display: flex;
  flex-direction: column;
}

label {
  color: #e0e0e0;
  margin-bottom: 5px;
  font-size: 16px;
}

input[type="text"] {
  padding: 10px;
  border: 1px solid #a020f0;
  border-radius: 5px;
  background-color: #2b2b2b;
  color: #e0e0e0;
  outline: none;
  font-size: 16px;
}

input[type="text"]:focus {
  border-color: #c71585;
}

.send-btn {
  padding: 12px;
  background-color: #a020f0;
  color: white;
  border: 2px solid white;
  border-radius: 5px;
  font-size: 18px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.send-btn:hover {
  background-color: #c71585;
}

.chat-box {
  margin-top: 20px;
  border: 1px solid #ccc;
  padding: 10px;
  height: 250px;
  overflow-y: auto;
  background-color: #2b2b2b;
  border-radius: 5px;
}

.message {
  margin-bottom: 3px;
  display: flex;
}

.message p {
  padding: 10px;
  border-radius: 10px;
  font-size: 16px;
  max-width: 100%;
}

.message-sent {
  background-color: #a020f0;
  color: white;
  align-self: flex-end;
}

.message-received {
  background-color: #444;
  color: white;
  align-self: flex-start;
}

@media (max-width: 600px) {
  .chat-container {
    flex-direction: column;
    align-items: center;
  }

  .chat {
    width: 100%;
  }

  .users-list {
    width: 100%;
    margin-top: 20px;
  }
}
</style>
