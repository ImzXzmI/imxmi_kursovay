<script setup>
import { ref, onMounted, defineProps, defineEmits } from "vue";

const props = defineProps({
  user: Object,
  orders: Array,
});

const emit = defineEmits(["logoutEmit", "goToFavorites", "RegistorEmit", "addOrder"]);

const showOrders = ref(false);
const showAvatarOptions = ref(false);

const username = ref("");
const password = ref("");
const email = ref("");
const selectedAvatar = ref(localStorage.getItem("selectedAvatar") || "/img/cat.PNG");

const errorUsername = ref(false);
const errorPassword = ref(false);
const errorEmail = ref(false);

const avatarOptions = [
  "/img/cat.PNG",
  "/img/cat2.PNG",
  "/img/cat3.PNG",
  "/img/cat4.PNG",
  "/img/cat5.PNG",
  "/img/cat6.PNG",
  "/img/cat7.PNG",
];

function toggleOrders() {
  showOrders.value = !showOrders.value;
}

function toggleAvatarOptions() {
  showAvatarOptions.value = !showAvatarOptions.value;
}

function selectAvatar(img) {
  selectedAvatar.value = img;
  localStorage.setItem("selectedAvatar", img);
  showAvatarOptions.value = false;
}

function check() {
  let passCheck = true;

  if (!/^[a-zA-Z0-9_]{3,}$/.test(username.value)) {
    errorUsername.value = true;
    passCheck = false;
  } else {
    errorUsername.value = false;
  }

  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value)) {
    errorEmail.value = true;
    passCheck = false;
  } else {
    errorEmail.value = false;
  }

  if (password.value.length < 6) {
    errorPassword.value = true;
    passCheck = false;
  } else {
    errorPassword.value = false;
  }

  if (passCheck) {
    emit("RegistorEmit", {
      username: username.value,
      password: password.value,
      email: email.value,
    });

    username.value = "";
    password.value = "";
    email.value = "";
  }
}

function handleLogout() {
  emit("logoutEmit");
}

function addOrder() {
  const newOrder = {
    date: new Date().toLocaleDateString(),
    firstName: props.user.firstName, // Используем данные пользователя
    lastName: props.user.lastName, // Используем данные пользователя
    email: props.user.email, // Используем email из данных пользователя
    address: props.user.address, // Используем данные пользователя
    comment: "Новый заказ", // Временно задаем комментарий
    delivery: "Курьером", // Временно задаем способ доставки
    payment: "Картой", // Временно задаем способ оплаты
    items: [
      { name: "Товар 1", quantity: 1, price: 1000 },
      { name: "Товар 2", quantity: 2, price: 500 },
    ],
    total: 2000,
  };
  emit("addOrder", newOrder);
}

onMounted(() => {
  const savedAvatar = localStorage.getItem("selectedAvatar");
  if (savedAvatar) {
    selectedAvatar.value = savedAvatar;
  }
});

// При монтировании компонента прокручиваем страницу вверх
onMounted(() => {
  window.scrollTo({ top: 0, behavior: 'smooth' }); // Плавная прокрутка вверх
});
</script>

<template>
  <div class="container">
    <div v-if="!props.user" class="form-box">
      <h1>Регистрация</h1>

      <label>
        Имя пользователя:
        <input type="text" v-model="username" placeholder="Например: user123" />
      </label>
      <div class="error" v-if="errorUsername">Имя должно быть минимум 3 символа и содержать только латиницу/цифры</div>

      <label>
        Email:
        <input type="email" v-model="email" placeholder="example@mail.com" />
      </label>
      <div class="error" v-if="errorEmail">Введите корректный email</div>

      <label>
        Пароль:
        <input type="password" v-model="password" placeholder="Не менее 6 символов" />
      </label>
      <div class="error" v-if="errorPassword">Пароль должен быть не менее 6 символов</div>

      <button class="btn" @click="check">Зарегистрироваться</button>
    </div>

    <!-- Личный кабинет -->
    <div v-else class="profile-box">
      <h1>Личный кабинет</h1>

      <div class="avatar-section">
        <h2>Аватар</h2>
        <img :src="selectedAvatar" alt="avatar" class="selected-avatar" /><br>
        <button class="btn small" @click="toggleAvatarOptions">
          {{ showAvatarOptions ? "Скрыть аватары" : "Изменить аватар" }}
        </button><br>

        <div v-if="showAvatarOptions" class="avatar-options">
          <img
            v-for="(img, index) in avatarOptions"
            :key="index"
            :src="img"
            class="avatar-option"
            :class="{ selected: selectedAvatar === img }"
            @click="selectAvatar(img)"
          />
        </div>
      </div>

      <p><strong>Имя пользователя:</strong> {{ props.user.username }}</p>
      <p><strong>Email:</strong> {{ props.user.email || "Не указан" }}</p>

      <div class="section">
        <h2 @click="toggleOrders" class="collapsible-title">
          {{ showOrders ? "▾ Скрыть заказы" : "▸ Мои заказы" }}
        </h2>
        <div v-if="showOrders" class="order-list">
          <div v-if="props.orders.length > 0">
            <div v-for="(order, index) in props.orders" :key="index" class="order-entry">
              <h3>Заказ №{{ index + 1 }} — {{ order.date }}</h3>
              <ul>
                <li><strong>Имя:</strong> {{ order.firstName }}</li>
                <li><strong>Фамилия:</strong> {{ order.lastName }}</li>
                <li><strong>Email:</strong> {{ order.email }}</li>
                <li><strong>Адрес:</strong> {{ order.address }}</li>
                <li><strong>Комментарий:</strong> {{ order.comment }}</li>
                <li><strong>Доставка:</strong> {{ order.delivery }}</li>
                <li><strong>Оплата:</strong> {{ order.payment }}</li>
              </ul>
              <ul>
                <li v-for="(item, i) in order.items" :key="i">
                  {{ item.name }} — {{ item.quantity }} × {{ item.price }} ₽ = {{ item.quantity * item.price }} ₽
                </li>
              </ul>
              <p><strong>Сумма:</strong> {{ order.total }} ₽</p>
              <hr />
            </div>
          </div>
          <p v-else>Заказов пока нет.</p>
        </div>
      </div>

      <button class="btn secondary" @click="$emit('goToFavorites')">Перейти в избранное</button>
      <button class="btn logout" @click="handleLogout">Выйти</button>
    </div>
  </div>
</template>

<style scoped>

/* Общий стиль для кнопок */
.btn {
  background-color: #e0e0e0;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  color: #333;
  transition: background-color 0.2s ease;
}

.btn:hover {
  background-color: #d0d0d0;
}

/* Меньшая версия кнопки, менее заметная */
.btn.small {
  width: 30%;
  background-color: transparent;
  color: #666;
  font-size: 13px;
  padding: 6px 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  align-items: flex-end;
}

.btn.small:hover {
  background-color: #d6a9fd28;
  color: #333;
}

.container {
  max-width: 35%;
  margin: 6% auto;
  padding: 24px;
  background: #f9f9f982;
  border-radius: 16px;
  box-shadow: 0 0 12px #D6A9FD;
  font-family: "Segoe UI", sans-serif;
  margin-top: 125px;
}

h1 {
  text-align: center;
  margin-bottom: 24px;
}

.form-box label {
  display: block;
  margin-bottom: 12px;
  font-weight: bold;
}

input[type="text"],
input[type="password"],
input[type="email"] {
  width: 100%;
  padding: 10px;
  margin-top: 4px;
  border: 1px solid #D6A9FD;
  border-radius: 8px;
  box-sizing: border-box;
}

.error {
  color: #d60000;
  font-size: 0.9em;
  margin-bottom: 12px;
}

.btn {
  width: 100%;
  padding: 12px;
  background-color: #D6A9FD;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1em;
  cursor: pointer;
  margin-top: 16px;
  transition: background 0.2s ease;
}

.btn:hover {
  background-color: #7f5e9c;
}

.btn.secondary {
  background-color: #D6A9FD;
}

.btn.secondary:hover {
  background-color: #7f5e9c;
}

.logout {
  background-color: #9AE3F3;
  margin-top: 16px;
}

.logout:hover {
  background-color: #4c8997;
}

.profile-box {
  text-align: center;
}

.section {
  margin-top: 20px;
  background: #ffffff87;
  padding: 12px;
  border: 1px solid #9AE3F3;
  box-shadow: 0 0 12px #9AE3F3;
  border-radius: 12px;
  text-align: left;
}

.collapsible-title {
  cursor: pointer;
  font-weight: bold;
  font-size: 1.1em;
  margin-bottom: 8px;
}

.order-list {
  padding-left: 10px;
}

.avatar-section {
  margin-top: 20px;
}

.selected-avatar {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  border: 3px solid #D6A9FD;
  margin-bottom: 10px;
}

.avatar-options {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
}

.avatar-option {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid transparent;
  transition: border 0.2s;
}

.avatar-option.selected {
  border-color: #D6A9FD;
  box-shadow: 0 0 6px #D6A9FD;
}

</style>
