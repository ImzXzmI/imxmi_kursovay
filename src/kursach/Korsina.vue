<script setup>
// Импортируем реактивные переменные и вычисляемые свойства из Vue
import { ref, computed } from "vue";

// Импортируем компоненты: модальное окно авторизации и форму оформления заказа
import AuthPromptModal from "./OrderModal.vue";
import Oformlenie from "./oformlenie.vue";

// Объявляем входные свойства (пропсы), передаваемые от родительского компонента
const props = defineProps({
  // Корзина с товарами
  cart: {
    type: Array,
    required: true,
  },
  // Текущий пользователь (объект)
  user: {
    type: Object,
    required: true,
  },
  // Список всех заказов
  orders: {
    type: Array,
    required: true,
  },
  // Список всех товаров (для отображения и проверки количества)
  product: Array
});


// Уведомление о продукте
const showNotification = ref(false);
const notificationMessage = ref("Товар добавлен в корзину!");

function toggleNotification() {
  console.log('local.products:', props.product);
  showNotification.value = !showNotification.value;
}

// Объявляем события, которые компонент может отправить родителю
const emit = defineEmits([
  "updateCart",   // Обновить содержимое корзины
  "openCatalog",  // Открыть каталог
  "addOrder",     // Добавить новый заказ
  "clearCart",    // Очистить корзину после оформления заказа
  "removeFromCart"
]);

// Управление отображением модального окна авторизации
const showModal = ref(false);

// Управление отображением формы оформления заказа
const oformlenie = ref(false);

// Вычисляем общую сумму заказа — суммируем цену каждого товара * его количество
const totalPrice = computed(() =>
  props.cart.reduce((sum, item) => sum + item.price * item.quantity, 0)
);


const errorMessage = ref('');


function updateQuantity(product, change) {
  console.log('local.products:', props.product);
  // Проверяем, что props.products существует и является массивом
  if (!Array.isArray(props.product)) {
    console.warn("Пропс products не передан или не является массивом.");
    return;
  }

  // Создаем копию корзины
  const updatedCart = props.cart.map(item => ({ ...item }));
  console.log('props.cart.map:', props.cart.map);

  // Находим индекс нужного товара по его id
  const index = updatedCart.findIndex(item => item.id === product.id);
  console.log('updatedCart.findIndex', index);

  // Если товар найден в корзине
  if (index !== -1) {
    // Текущее количество товара в корзине
    const currentCount = updatedCart[index].quantity;
    console.log('currentCount ', currentCount);

    // Проверяем, не превышает ли новое количество доступное количество на складе
    const productInStock = props.product.find(item => item.id === product.id);
    console.log('productInStock ', productInStock);
    if (productInStock && currentCount + change <= productInStock.quantity) {
      // Изменяем количество товара в корзине
      updatedCart[index].quantity += change;

      // Если количество стало меньше или равно 0 — удаляем товар из корзины
      if (updatedCart[index].quantity <= 0) {
        updatedCart.splice(index, 1);
      }

      // Отправляем обновленную корзину родителю
      emit("updateCart", updatedCart);
      errorMessage.value = ''; // Очистить сообщение об ошибке при успешной операции
    } else {
      // Если количество в корзине превышает доступное количество на складе
      console.warn(`Невозможно добавить больше ${product.quantity} шт. товара "${product.name}" в корзину.`);
      errorMessage.value = `Невозможно добавить больше ${productInStock.quantity} шт. товара "${product.name}" в корзину.`;
    }
  }
}


// Функция запуска оформления заказа
function placeOrder() {
  // Если пользователь авторизован — открываем форму оформления заказа
  if (props.user) {
    oformlenie.value = true;
  } else {
    // Иначе показываем модальное окно с предложением войти
    showModal.value = true;
  }
}

// Функция обработки данных формы оформления заказа
function handleFormSubmit(formData) {
  // Создаём объект нового заказа
  const newOrder = {
    ...formData,                      // Данные из формы (например, имя, адрес)
    items: [...props.cart],          // Текущие товары в корзине
    total: totalPrice.value,         // Общая сумма заказа
    date: new Date().toLocaleString() // Текущая дата и время оформления
  };

  // Отправляем заказ родителю
  emit("addOrder", newOrder);

  // Очищаем корзину после оформления
  emit("clearCart");

  // Закрываем форму оформления
  oformlenie.value = false;
}

function remove(product) {
  // Эмитим событие с продуктом, который нужно удалить
  emit("removeFromCart", product);
}

</script>



<template>
  <div class="cart-page">
    <h1>Корзина</h1>
    <div>


    <!-- Уведомление о добавлении товара -->
    <div v-if="showNotification" class="notification">
      <p>{{ notificationMessage }}</p>
      <button @click="showNotification = false">Закрыть</button>
    </div>
  </div>

    <!-- Текущая корзина -->
    <div v-if="cart.length > 0">
      <ul class="cart-list">
        <li v-for="(product, index) in cart" :key="index" class="cart-item">
          <img :src="product.images[0]" :alt="product.name" class="cart-image" />
          
          <div class="cart-info">
            <p class="cart-name">{{ product.name }}</p>
            <p class="cart-price">{{ product.price }} ₽</p>
            <div class="cart-quantity">
              <button 
                @click="updateQuantity(product, -1)" 
                :disabled="product.quantity <= 1"
              >
                −
              </button>
              <span>{{ product.quantity }}</span>
              <button 
                @click="updateQuantity(product, 1)" 
                :disabled="product.quantity >= product.stock"
              >
                +
              </button>
            </div>
            <p class="cart-total">Сумма: {{ product.price * product.quantity }} ₽</p>
          </div>
          <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
          <div>
<button @click="remove(product)" class="delete-button">
    Удалить
  </button>
  </div>
        </li>
      </ul>

      <div class="total-price">
        <h2>Общая сумма: {{ totalPrice }} ₽</h2>
      </div>

      <button class="catalog-button" @click="placeOrder">Оформить заказ</button>
    </div>

    <div v-else-if="cart.length === 0">
      <p>Ваша корзина пуста.</p>
    </div>

    <button class="order-button" @click="$emit('openCatalog')">Перейти в каталог</button>

    <Oformlenie
      v-if="oformlenie"
      @close="oformlenie = false"
      @updateForm="handleFormSubmit"
    />

    <AuthPromptModal
      v-if="showModal"
      @close="showModal = false"
      :openRegistrationPage="props.openRegistrationPage"
    />
  </div>
  
</template>

<style scoped>
/* Стили для кнопки "Удалить" */
.delete-button {
  padding: 10px 20px;
  background-color: #f5a8b5; /* Светло-розовый фон */
  color: white;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

/* Эффект при наведении */
.delete-button:hover {
  background-color: #f18498; /* Более насыщенный розовый */
  transform: scale(1.05);
}

/* Эффект при нажатии */
.delete-button:active {
  background-color: #e06b8a; /* Темнее розовый при нажатии */
  transform: scale(0.98);
}

/* Добавление тени для кнопки */
.delete-button:focus {
  outline: none;
  box-shadow: 0 0 5px rgba(255, 105, 180, 0.4); /* Светлая розовая тень при фокусе */
}
.cart-page {
  padding: 20px;
  font-family: 'Arial', sans-serif;
  margin-top: 60px;
}

.cart-list {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  padding: 0;
  list-style: none;
}

.cart-item {
  border: 1px solid #c5cae9;
  border-radius: 16px;
  padding: 16px;
  width: 240px;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: transform 0.3s;
}

.cart-item:hover {
  transform: scale(1.03);
}

.cart-image {
  width: 100%;
  height: 140px;
  object-fit: cover;
  border-radius: 12px;
  margin-bottom: 10px;
}

.cart-info {
  text-align: center;
}

.cart-name {
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 6px;
  color: #4a148c;
}

.cart-price,
.cart-quantity,
.cart-total {
  font-size: 14px;
  color: #4e4e4e;
  margin: 4px 0;
}

.cart-quantity {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.cart-quantity button {
  margin: 0;
  padding: 6px 12px;
  font-size: 14px;
  border: none;
  background-color: #ce93d8;
  color: white;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
}

.cart-quantity button:hover {
  background-color: #ab47bc;
}

.cart-total {
  font-size: 16px;
  font-weight: 600;
  margin-top: 10px;
}

.total-price {
  margin-top: 20px;
  text-align: center;
}

.error-message {
  color: red;
  font-size: 14px;
  margin-top: 5px;
}


.order-button {
  margin-top: 20px;
  padding: 10px 20px;
  box-shadow: 0 4px 12px #ce93d8;
  background-color: #f7cbff;
  border: solid 1px #ce93d8;
  color: rgb(0, 0, 0);
  font-weight: bold;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: background 0.3s;
}

.order-button:hover {
  background-color: #ce93d8;
}

.catalog-button {
  margin-top: 16px;
  padding: 10px 20px;
  background-color: #5c6bc0;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: background 0.3s;
}

.catalog-button:hover {
  background-color: #3f51b5;
}

</style>

