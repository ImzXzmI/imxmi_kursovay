<script setup>
import { ref, reactive, computed } from "vue";
import Glavnay from "./Glavnay.vue";
import Katolog1 from "./Katolog1.vue";
import Katolog2 from "./Katolog2.vue";
import Dostavka from "./Dostavka.vue";
import Korsina from "./Korsina.vue";
import LK from "./LK.vue";
import Izbrannoe from "./Izbrannoe.vue";
import OrderModal from "./OrderModal.vue";
import Oformlenie from "./oformlenie.vue";

const isModalOpen = ref(false);
const showSidebar = ref(false);
const selectedAvatar = ref("https://cdn-icons-png.flaticon.com/512/9131/9131529.png");

const local = reactive({
  curPage: 1,
  user: JSON.parse(localStorage.getItem("user")) || null,
  cart: JSON.parse(localStorage.getItem('cart')) || [],
  favorites: JSON.parse(localStorage.getItem("favorites")) || [],
  form: JSON.parse(localStorage.getItem("form")) || null,
  orders: JSON.parse(localStorage.getItem("orders")) || [],
  products: [  
  {
    id: 1,
    name: "Ожерелье «Love crime»",
    category: "ожерелье",
    price: 3000,
    quantity: 10,
    images: ["/img/j1.1.jpg", "/img/j1.2.jpg", "/img/j1.3.jpg"],
    description: "Элегантное ожерелье, вдохновлённое сериалом Ганнибал.",
    reviews: ["Отличный товар!", "Качественный!"]
  },
  {
    id: 2,
    name: "Ожерелье «Rinascimento»",
    category: "ожерелье",
    price: 3000,
    quantity: 7,
    images: ["/img/j2.1.jpg", "/img/j2.2.jpg", "/img/j2.3.jpg"],
    description: "Изящное украшение красными камнями, вдохновлённое Италие в Эпоху Возрождения.",
    reviews: ["Очень нравится!", "Рекомендую!"]
  },
  {
    id: 3,
    name: "Ожерелье «The starry sky»",
    category: "ожерелье",
    price: 2500,
    quantity: 5,
    images: ["/img/j3.1.jpg", "/img/j3.2.jpg", "/img/j3.3.jpg"],
    description: "Яркое и загадочное ожерелье с переливающимися камнями, каждый из них блестит, словно зведное небо.",
    reviews: ["Хорошее качество", "Доволен покупкой"]
  },
  {
    id: 4,
    name: "Браслет «The rebirth of the villainess»",
    category: "браслет",
    price: 5000,
    quantity: 1,
    unique: true,
    images: ["/img/j4.1.jpg"],
    description: "Эксклюзивный браслет ручной работы с натуральным топазом. <strong>Единственный экземпляр!</strong>",
    reviews: ["Хорошее качество", "Доволен покупкой"]
  },
  {
    id: 5,
    name: "Ожерелье «Elven Queen»",
    category: "ожерелье",
    price: 5000,
    quantity: 3,
    images: ["/img/j5.1.jpg", "/img/j5.2.jpg", "/img/j5.3.jpg"],
    description: "Волшебное поистине красивое ожерелье",
    reviews: ["Хорошее качество", "Доволен покупкой"]
  }
]




});

const authButtonText = computed(() => (local.user ? local.user.username : "Войти"));
const isAuthenticated = computed(() => !!local.user);



// Функция для добавления нового заказа
function handleOrderAddition(newOrder) {
  // Добавляем новый заказ в массив
  local.orders.push(newOrder);

  localStorage.setItem("orders", JSON.stringify(local.orders));
}

function registerUser(data) {
  local.user = data;
  localStorage.setItem("user", JSON.stringify(local.user));
}

function logoutUser() {
  local.user = null;
  local.orders = []; // Обнуляем массив заказов в local
  localStorage.removeItem("user");
  localStorage.removeItem("orders"); // Удаляем заказы из localStorage
}

function addToCart(product) {
  const existing = local.cart.find((item) => item.id === product.id);
  const availableProduct = local.products.find((item) => item.id === product.id);
  const availableQuantity = availableProduct ? availableProduct.quantity : 0;

  if (existing) {
    const newQuantity = existing.quantity + 1;
    if (newQuantity <= availableQuantity) {
      existing.quantity = newQuantity;
    } else {
      console.warn(`Невозможно добавить больше ${availableQuantity} шт. товара "${product.name}" в корзину.`);
    }
  } else {
    if (availableQuantity > 0) {
      local.cart.push({ ...product, quantity: 1 });
    } else {
      console.warn(`Товара "${product.name}" нет в наличии.`);
    }
  }

  // Сохраняем корзину в localStorage
  localStorage.setItem('cart', JSON.stringify(local.cart));
}


function removeFromCart(product) {
  const index = local.cart.findIndex((item) => item.id === product.id);
  if (index !== -1) {
    if (local.cart[index].quantity > 1) {
      local.cart[index].quantity--;
    } else {
      local.cart.splice(index, 1);
    }
  }
}

function toggleFavorite(product) {
  const index = local.favorites.findIndex((item) => item.id === product.id);
  if (index !== -1) {
    local.favorites.splice(index, 1);
  } else {
    local.favorites.push(product);
  }
  localStorage.setItem("favorites", JSON.stringify(local.favorites));
}

function isFavorite(productId) {
  return local.favorites.some((item) => item.id === productId);
}

const cartGrouped = computed(() =>
  local.cart.map((item) => ({
    id: item.id,
    name: item.name,
    price: item.price,
    quantity: item.quantity,
    totalPrice: item.price * item.quantity,
  }))
);

const totalAmount = computed(() =>
  cartGrouped.value.reduce((sum, item) => sum + item.totalPrice, 0)
);

// Обновление данных формы и создание заказа
function updateForm(data) {
  local.form = { ...data };
  localStorage.setItem("form", JSON.stringify(local.form));

  const newOrder = {
    id: Date.now(),
    items: [...local.cart],
    info: { ...data },
    total: totalAmount.value,
    date: new Date().toLocaleString()
  };

  handleOrderAddition(newOrder); // Используем функцию добавления заказа
  local.cart = [];
}

// Переходы
function openCart() {
  local.curPage = 5;
}
function openCatalog() {
  local.curPage = 2;
  window.scrollTo({ top: 0, behavior: "smooth" });
}
function handleCloseModal() {
  isModalOpen.value = false;
}
function goToRegister() {
  local.curPage = 6;
  isModalOpen.value = false;
}
function goToCheckout() {
  local.curPage = 8;
  isModalOpen.value = false;
}


</script>

<template>

  <div class="app-wrapper">



    <div class="main-content">
  <div class="header">
    <div class="panel-open">
      <div class="menu">
        <!-- Пункт меню с логотипом -->
        <button @click="local.curPage = 1" >
          <img src="/img/logo1.png" alt="Logo" class="logo" />
        </button>

        <button @click="local.curPage = 1" :class="{ menuIt: local.curPage == 1 }">О нас</button>
        <button @click="openCatalog" :class="{ menuIt: local.curPage == 2 }">Каталог</button>
        <button @click="local.curPage = 4" :class="{ menuIt: local.curPage == 4 }">Доставка</button>
        <button @click="openCart" :class="{ menuIt: local.curPage == 5 }">Корзина</button>
        <button @click="local.curPage = 6" :class="{ menuIt: local.curPage == 6, 'login-btn': !local.user }">
          {{ authButtonText }}
        </button>

        <P @click="local.curPage = 7" class="heart-button">
          ❤️
        </P>
      </div>
    </div>
  </div>



      <Glavnay v-if="local.curPage == 1" 
      @openCatalog="openCatalog"
      :products="local.products"
        @addToCart="addToCart"
        @toggleFavorite="toggleFavorite"
        :isFavorite="isFavorite"
        @openCart="openCart"
        :cart="local.cart"
        @removeFromCart="removeFromCart"/>
      <Katolog1 v-if="local.curPage == 2"
        :products="local.products"
        @addToCart="addToCart"
        @toggleFavorite="toggleFavorite"
        :isFavorite="isFavorite"
        @openCart="openCart"
        :cart="local.cart"
        @removeFromCart="removeFromCart"
      />
      <Katolog2 v-if="local.curPage == 3" />
      <Dostavka v-if="local.curPage == 4" />
      <Korsina v-if="local.curPage == 5"
  :cart="local.cart"
  :product="local.products"  
  :user="local.user"
  :orders="local.orders"
  @updateCart="(newCart) => local.cart = newCart"
  @removeFromCart="removeFromCart"
  @openCatalog="openCatalog"
  @toggleFavorite="toggleFavorite"
  @updateForm="updateForm"
  @clearCart="() => local.cart = []"
  @addOrder="handleOrderAddition" 
/>
      <LK v-if="local.curPage == 6"
        :user="local.user"
        :form="local.form"
        :orders="local.orders"
        @RegistorEmit="registerUser"
        @logoutEmit="logoutUser"
        @goToFavorites="local.curPage = 7"
        @addOrder="handleOrderAddition" 
      />
      <Izbrannoe v-if="local.curPage == 7"
        :favorites="local.favorites"
        @toggleFavorite="toggleFavorite"
        @addToCart="addToCart"
        @openCatalog="openCatalog"
      />
      <Oformlenie 
        v-if="local.curPage == 8"
        :form="local.form"
        @updateForm="updateForm"
        @clearCart="() => local.cart = []"
        @close="local.curPage = 5"
      />
      <OrderModal
        v-if="isModalOpen"
        :isAuthenticated="isAuthenticated"
        @close="handleCloseModal"
        @goToRegister="goToRegister"
        @goToCheckout="goToCheckout"
      />
    </div>
  </div>
</template>


<style scoped>
/* Базовая структура */

.logo {
  width: 70px; /* Устанавливает ширину логотипа */
  height: auto; /* Поддерживает пропорции */

}

.app-wrapper {
  display: flex;
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(to bottom, white, #e0f7fa 50%, #d1c4e9 100%);
  margin: 0;
  padding: 0;
}

.selected {
  border-color: #5e35b1;
}

/* Корзина */
ul {
  list-style-type: none;
  padding-left: 0;
}

ul li {
  margin: 10px 0;
}

.order-button {
  padding: 10px 20px;
  background-color: #7e57c2;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.order-button:hover {
  background-color: #5e35b1;
}

/* Основной контент */
.main-content {
  flex: 1;
}

.header {
  width: 100%;
  display: flex;
  justify-content: center;
}

/* Меню */
.menu {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: rgba(237, 231, 246, 0.6);

  backdrop-filter: blur(8px);
  box-shadow: 0 6px 20px rgba(126, 87, 194, 0.2);
  width: 100%;
  transition: 0.4s ease;
}

/* Контейнер для кнопок и сердца */
.menu-buttons {
  display: flex;
  gap: 12px;
  align-items: center;
}

/* Кнопки меню */
.menu button {
  flex: 1;
  text-align: center;
  padding: 10px 16px;
  margin: 0 6px;
  background-color: transparent;
  color: #4a148c;
  font-weight: 600;
  font-size: 15px;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: 0.3s ease;
  white-space: nowrap;
}

.menu button:hover {
  background-color: rgba(149, 117, 205, 0.25);
  color: #311b92;
  transform: translateY(-2px);
}

.menuIt {
  background-color: #7e57c2;
  color: white;
  box-shadow: 0 0 10px rgba(126, 87, 194, 0.6);
  font-weight: bold;
}

/* Кнопка входа */
.login-btn {
  background-color: rgba(171, 71, 188, 0.15) !important;
  color: #6a1b9a !important;
}

.login-btn.menuIt {
  background-color: rgba(171, 71, 188, 0.4) !important;
  color: #4a148c !important;
  box-shadow: 0 0 6px rgba(171, 71, 188, 0.5) !important;
}

.login-btn:hover {
  background-color: rgba(171, 71, 188, 0.3) !important;
}

.panel-open {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;

  position: relative;
}

/* Сердце сдвинуто вправо */
.heart-button {
  font-size: 24px;
  background: none;
  border: none;
  cursor: pointer;
  color: red;
  transition: transform 0.3s ease-in-out;
  margin-right: 60px;
}

/* Пульсация при наведении */
.heart-button:hover {
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
  }
}
</style>
