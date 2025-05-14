<script setup>
import { ref, computed, onMounted } from "vue";

const props = defineProps({
  products: Array,
  isFavorite: Function,
  cart: Array
});

const emit = defineEmits([
  "addToCart",
  "removeFromCart",
  "openCart",
  "toggleFavorite"
]);

const selectedProduct = ref(null);
const cartCounts = ref({});  // Храним количество товаров в корзине
const selectedCategory = ref("все");
const searchTerm = ref('');
const fullScreenImage = ref(null);
const fullScreenIndex = ref(null);

// Открытие изображения на весь экран
function openFullScreen(image) {
  const index = selectedProduct.value.images.indexOf(image);
  if (index !== -1) {
    fullScreenIndex.value = index;
  }
}

function closeFullScreen() {
  fullScreenIndex.value = null;
}

function nextImage() {
  if (fullScreenIndex.value !== null) {
    fullScreenIndex.value =
      (fullScreenIndex.value + 1) % selectedProduct.value.images.length;
  }
}

function prevImage() {
  if (fullScreenIndex.value !== null) {
    fullScreenIndex.value =
      (fullScreenIndex.value - 1 + selectedProduct.value.images.length) % 
      selectedProduct.value.images.length;
  }
}

// Получение категорий
const categories = computed(() => {
  const cats = new Set(props.products.map((p) => p.category));
  return ["все", ...cats];
});

const showFilters = ref(false);
const minPrice = ref(null);
const maxPrice = ref(null);

// Фильтрация товаров
const filteredProducts = computed(() => {
  return props.products.filter((product) => {
    const inCategory =
      selectedCategory.value === "все" ||
      product.category === selectedCategory.value;
    const inMinPrice =
      minPrice.value == null || product.price >= minPrice.value;
    const inMaxPrice =
      maxPrice.value == null || product.price <= maxPrice.value;
    const matchesSearchTerm = product.name
      .toLowerCase()
      .includes(searchTerm.value.toLowerCase());

    return inCategory && inMinPrice && inMaxPrice && matchesSearchTerm;
  });
});

// Применение фильтров
function applyFilters() {
  showFilters.value = false;
}

// Сброс фильтров
function resetFilters() {
  selectedCategory.value = "все";
  minPrice.value = null;
  maxPrice.value = null;
  searchTerm.value = '';
}

// При монтировании инициализируем cartCounts
onMounted(() => {
  props.cart.forEach((item) => {
    cartCounts.value[item.id] = item.quantity;
  });
});

function closeProduct() {
  selectedProduct.value = null;
}

function goToCart() {
  emit("openCart");
}

function incrementProduct(product) {
  const currentCount = cartCounts.value[product.id] || 0;

  // Проверяем, не превышает ли новое количество доступное
  if (currentCount < product.quantity) {
    const newCount = currentCount + 1;
    cartCounts.value[product.id] = newCount;

    const updatedProduct = {
      ...product,
      quantity: newCount
    };

    emit("addToCart", updatedProduct);
  } else {
    // Опционально: можно вывести предупреждение или заблокировать кнопку
    console.warn(`Нельзя добавить больше ${product.quantity} шт. товара "${product.name}"`);
  }
}


// ✅ Уменьшение количества товара в корзине
function decrementProduct(product) {
  const currentCount = cartCounts.value[product.id] || 0;

  if (currentCount > 1) {
    const newCount = currentCount - 1;
    cartCounts.value[product.id] = newCount;

    const updatedProduct = {
      ...product,
      quantity: newCount
    };

    emit("removeFromCart", updatedProduct);
  } else if (currentCount === 1) {
    delete cartCounts.value[product.id];

    const updatedProduct = {
      ...product,
      quantity: 0
    };

    emit("removeFromCart", updatedProduct);
  }
}

// Избранное
function handleToggleFavorite(product) {
  emit("toggleFavorite", product);
}

const currentImageIndex = ref(0);

// Стили галереи
const galleryStyle = computed(() => ({
  display: "grid",
  gridTemplateColumns: "repeat(auto-fit, minmax(200px, 1fr))",
  gap: "16px",
  overflowX: "auto"
}));

</script>



<template>
  <div class="catalog">
    <!-- Модальное окно с подробной информацией -->

    <div v-if="selectedProduct" class="product-detail-modal" >
      <div class="overlay" @click="closeProduct"></div>
      <div class="product-detail">
        <button class="back-button" @click="closeProduct">Назад</button>
          <!-- КНОПКА ИЗБРАННОГО В ПРАВОМ ВЕРХУ -->
  <button
    @click.stop="handleToggleFavorite(selectedProduct)"
    class="favorite-button-top"
    :title="isFavorite(selectedProduct.id) ? 'Удалить из избранного' : 'Добавить в избранное'"
  >
    {{ isFavorite(selectedProduct.id) ? '💔' : '❤️' }}
  </button>

        <div class="product-images">
          <div class="image-selector">
            <img
  v-for="(image, index) in selectedProduct.images"
  :key="index"
  :src="image"
  :alt="'Фото товара ' + (index + 1)"
  :class="['thumbnail', { 'active': index === currentImageIndex }]"
  @click.stop="openFullScreen(image)"
/>
          </div>
          <img
            :src="selectedProduct.images[currentImageIndex]"
            :alt="selectedProduct.name"
            class="large-image"
           @click.stop="openFullScreen(selectedProduct.images[currentImageIndex])"
          />

          <div v-if="fullScreenIndex !== null" class="fullscreen-image-modal">
  <button class="nav-button prev" @click.stop="prevImage">◀</button>
  <img :src="selectedProduct.images[fullScreenIndex]" class="fullscreen-image" @click.stop />
  <button class="nav-button next" @click.stop="nextImage">▶</button>
  <button class="close-button" @click="closeFullScreen">✕</button>
</div>
        </div>

        <h2>{{ selectedProduct.name }}</h2>
        <p><strong>Цена:</strong> {{ selectedProduct.price }} ₽</p>
        <div v-if="cartCounts[selectedProduct.id]">
          <div class="cart-controls">
            <button @click.stop="decrementProduct(selectedProduct)">➖</button>
            <button @click.stop="goToCart">🛒 {{ cartCounts[selectedProduct.id] }} шт.</button>
            <button @click.stop="incrementProduct(selectedProduct)">➕</button>
          </div>
          <p><strong>В наличии:</strong> {{ selectedProduct.quantity }} шт.</p>
        </div>
        
        <button
          v-else
          @click.stop="incrementProduct(selectedProduct)"
          class="add-to-cart-button"
        >
          Добавить в корзину
        </button>
        <p>{{ selectedProduct.description }}</p>
    



        <button @click.stop="handleToggleFavorite(selectedProduct)" class="favorite-button">
          {{ isFavorite(selectedProduct.id) ? '💔 Удалить из избранного' : '❤️ В избранное' }}
        </button>
      </div>
      </div>
    </div>
    <!-- ФИЛЬТР -->
  
  
    <div class="zagolovok2"><p class="zagolovok3">Кольчужные украшения</p></div>
    <div class="search-container">
      <div class="zagolovok1"><p class="zagolovok">ImxmI</p></div>
      <div class="search-container">
  <input 
    type="text" 
    v-model="searchTerm" 
    placeholder="Поиск украшения..." 
    class="search-input" 
  />
</div>
</div>
<div class="knopka">
  <div class="zagolovok1"><P class="zagolovok">Найди украшение для себя!</p></div>
    <div class="knopka"><button class="toggle-filters-button" @click="showFilters = !showFilters">
      {{ showFilters ? 'Скрыть фильтры' : 'Фильтры' }}
    </button>
  </div></div>

  

  <DIV class="CENTER">

    <div v-if="showFilters" class="filters">
      <div class="category-filters">
        <button
          v-for="category in categories"
          :key="category"
          :class="{ active: selectedCategory === category }"
          @click="selectedCategory = category"
        >
          {{ category }}
        </button>
      </div>

      <div class="price-filters">
        <label>
          От:
          <input type="number" v-model.number="minPrice" placeholder="мин. цена" />
        </label>
        <label>
          До:
          <input type="number" v-model.number="maxPrice" placeholder="макс. цена" />
        </label>
      </div>

      <button class="apply-button" @click="applyFilters">Применить</button>
      <button class="reset-button" @click="resetFilters">Сбросить фильтры</button>
    </div>
  </DIV>
  <div class="content-wrapper" :class="{ 'filters-open': showFilters }">


      <!-- Каталог товаров -->
  <div class="product-list">
    <div v-if="filteredProducts.length === 0" class="no-products">
      <p>Нет подходящих товаров</p>
    </div>
    <div
      v-for="product in filteredProducts"
      :key="product.id"
      class="product-card"
      @click="selectedProduct = product"
    >
      <button @click.stop="handleToggleFavorite(product)" class="favorite-button">
        {{ isFavorite(product.id) ? '💔' : '❤️' }}
      </button>
      <img :src="product.images[0]" :alt="product.name" class="product-image" />
      <h3 class="product-name">{{ product.name }}</h3>
      <p class="product-price">{{ product.price }} ₽</p>
      <p v-if="product.quantity === 1" class="limited-tag">Единственный экземпляр</p>
      <p class="product-quantity">В наличии: {{ product.quantity }} шт.</p>
      <div v-if="cartCounts[product.id]">
        <div class="cart-controls">
          <button @click.stop="decrementProduct(product)">➖</button>
          <button @click.stop="goToCart">🛒 {{ cartCounts[product.id] }} шт.</button>
          <button @click.stop="incrementProduct(product)">➕</button>
        </div>
      </div>
      <button
        v-else
        @click.stop="incrementProduct(product)"
        class="add-to-cart-button"
      >
        Добавить в корзину
      </button>
    </div>
  </div>
    <div v-if="showFilters" class="filters">
      <div class="category-filters">
        <button
          v-for="category in categories"
          :key="category"
          :class="{ active: selectedCategory === category }"
          @click="selectedCategory = category"
        >
          {{ category }}
        </button>
      </div>

      <div class="price-filters">
        <label>
          От:
          <input type="number" v-model.number="minPrice" placeholder="мин. цена" />
        </label>
        <label>
          До:
          <input type="number" v-model.number="maxPrice" placeholder="макс. цена" />
        </label>
      </div>

      <button class="apply-button" @click="applyFilters">Применить</button>
      <button class="reset-button" @click="resetFilters">Сбросить фильтры</button>
    </div>
  



      
    </div>

    <button @click="goToCart" class="cart-button">Перейти в корзину</button>

</template>

<style scoped>
/* Основной контейнер для каталога товаров */

.limited-tag {
  color: crimson;
  font-weight: bold;
}
.product-quantity {
  font-size: 0.9rem;
  color: #555;
}


.apply-button, .reset-button{
  margin: 15px;
}
.catalog {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  margin: 20 auto;
  margin-top: 60PX;
  width: 100%;
  max-width: 1200px;
  box-sizing: border-box;
}

/* Сеточная структура для карточек товаров */
.product-list {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* максимум 3 в ряд */
  gap: 24px;
  width: 100%;
  max-width: 1000px;
  box-sizing: border-box;
  margin: 0 auto;
  overflow-x: hidden; /* Убрали горизонтальный скроллинг */
}

/* Карточка товара */
.product-card {
  background: linear-gradient(to bottom, #ffffff, #e1f5fe);
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(126, 87, 194, 0.2);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px;
  max-width: 299px;
  width: 100%;
  cursor: pointer;
  position: relative; /* Для использования абсолютного позиционирования */
  display: flex;
  display: flex;
  flex-direction: column; /* Расположить элементы по вертикали */
  justify-content: space-between; /* Равномерно распределить пространство между элементами */
  min-height: 200px; /* Убедитесь, что карточка имеет достаточную высоту */
  border: 1px solid #ddd;
  padding: 16px;
  border-radius: 8px;
  text-align: center;
  cursor: pointer;
  
}

.product-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 8px 20px rgba(126, 87, 194, 0.3);

}

.product-image {
  width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 12px;
  margin-bottom: 10px;
}

.product-name {
  font-size: 18px;
  color: #4a148c;
  font-weight: 600;
  text-align: center;
  margin: 6px 0;
}

.product-price {
  font-size: 16px;
  color: #000000;
  margin-bottom: 10px;
}



.cart-controls {
  display: flex;
  gap: 10px;
  margin-top: auto; /* Прижимаем к низу карточки */
}


.add-to-cart-button {
  background-color: #9AE3F3;
  box-shadow: 0 0 12px #bc9af3;
  color: #4a148c;
  padding: 10px 16px;
  font-size: 14px;
  font-weight: bold;
  border-radius: 20px;
  border: solid 1PX #f7a9fd;
  cursor: pointer;
  margin-top: 10px;
  width: 100%;
}

.add-to-cart-button:hover {
  background-color: #9abbf3;
}

.favorite-container {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 10;
}

.favorite-button {
  background-color: transparent;
  border: none;
  color: #333;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.2s ease;
  position: relative;
}

.favorite-button::after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 120%;
  height: 120%;
  background-color: rgba(255, 0, 128, 0.2);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  opacity: 0;
  animation: pulse 0.8s infinite;
  z-index: -1;
}

@keyframes pulse {
  0% {
    transform: translate(-50%, -50%) scale(0.9);
    opacity: 0.6;
  }
  50% {
    transform: translate(-50%, -50%) scale(1.3);
    opacity: 0;
  }
  100% {
    transform: translate(-50%, -50%) scale(0.9);
    opacity: 0;
  }
}

.favorite-button:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

/* Модальное окно с подробной информацией */
/* Модальное окно с подробной информацией */
.product-detail-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  justify-content: center;
  align-items: center; /* Центрирование по вертикали и горизонтали */
  z-index: 100;
}

/* Стили для блока с содержимым */
.product-detail {
  background: linear-gradient(to bottom, #f3e5f5, #e1f5fe);
  padding: 20px;
  max-width: 500px;
  width: 100%;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  max-height: 75vh;
  overflow-y: auto;
  scrollbar-width: thin;               /* Firefox */
  scrollbar-color: #b39ddb transparent; /* Firefox */
  margin-top: 100PX;
}

/* Стилизация скроллбара для WebKit (Chrome, Edge, Safari) */
.product-detail::-webkit-scrollbar {
  width: 8px;
  border-radius: 10px;
}

.product-detail::-webkit-scrollbar-track {
  background: linear-gradient(to bottom, #f3e5f5, #e1f5fe);
  border-radius: 10px;
}

.product-detail::-webkit-scrollbar-thumb {
  background: linear-gradient(to bottom, #9575cd, #64b5f6);
  border-radius: 10px;
  border: 2px solid transparent;
  background-clip: content-box;
}

.large-image {
  width: 100%;
  height: 300px;
  object-fit: cover;
  border-radius: 12px;
  margin-bottom: 20px;
}

.image-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.thumbnail {
  width: 50px;
  height: 50px;
  object-fit: cover;
  border-radius: 5px;
  cursor: pointer;
  border: 2px solid transparent;
  transition: border 0.3s;
}

.thumbnail.active {
  border: 2px solid #D6A9FD;
}

/* Кнопка "Назад" */
.back-button {
  position: absolute;
  top: 16px;
  left: 20px;
  background-color: transparent;
  color: #7e57c2;
  font-weight: bold;
  font-size: 16px;
  border: none;
  cursor: pointer;
}

.back-button:hover {
  text-decoration: underline;
}

/* Кнопка сброса фильтров */
.reset-button {
  background-color: #9AE3F3;
  color: white;
  padding: 8px 16px;
  border: none;
  cursor: pointer;
  margin-top: 10px;
  font-size: 14px;
  border-radius: 5px;
}

.reset-button:hover {
  background-color: #499fb3;
}

/* Сообщение "Нет подходящих товаров" */
.no-products {
  text-align: center;
  font-size: 18px;
  color: #999;
  margin-top: 20px;
}

/* Стили для фильтров */
.filters {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-items: center;
  justify-content: center;
  gap: 1rem;
  background-color: #f9f9f971;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(166, 254, 255, 0.927);
  margin-bottom: 2rem;
  width: 100%;
  max-width: 300px;
  max-height: 400px;
  margin-right: 20px;

}

.category-filters,
.price-filters {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}



.category-filters button.active,
.category-filters button:hover {
  background-color: #8e44ad;
  color: #fff;
  margin: 10px;
  transition: all 1s;
}

.price-filters label {
  font-size: 0.9rem;
  color: #333;
}

.price-filters input {
  padding: 0.4rem 0.6rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 1rem;
  margin-top: 0.3rem;
  transition: border-color 0.2s;
}

.price-filters input:focus {
  border-color: #8e44ad;
  outline: none;
}

.apply-button {
  background-color: #D6A9FD;
  color: #fff;
  border: none;
  padding: 0.6rem 1.4rem;
  border-radius: 12px;
  cursor: pointer;
  transition: background-color 0.3s;
  width: 100%;
}

.apply-button:hover {
  background-color: #8e44ad;
}

@media (max-width: 960px) {
  .product-list {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 640px) {
  .product-list {
    grid-template-columns: 1fr;
  }
}

/* Модальное окно */
.fullscreen-image-modal {
  position: fixed;
  top: 50px; /* отступ под меню */
  left: 0;
  width: 100vw;
  height: calc(100vh - 80px); /* чтобы не перекрывало меню */
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: flex-start; /* расположение фото вверху */

  padding: 40px 20px 20px; /* отступы, чтобы не перекрывать края */
  box-sizing: border-box;
}

/* Установка изображения в модальном окне */
.fullscreen-image-modal img.fullscreen-image {
  max-width: 80vw;
  max-height: 80vh;
  object-fit: contain;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  margin-top: 20px; /* отступ сверху, чтобы фото не было у самого верха */
}
.fullscreen-image-modal .nav-button {
  position: absolute;
  padding: 12px;
  font-size: 28px;
  cursor: pointer;
  border-radius: 50%;
  z-index: 10000;
  background-color: rgba(255, 255, 255, 0.1); /* лёгкий прозрачный белый фон */
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.6); /* тонкая белая рамка */
  transition: background-color 0.3s ease, border-color 0.3s ease;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2); /* мягкая тень */
}

.fullscreen-image-modal .nav-button:hover {
  background-color: rgba(255, 255, 255, 0.2); /* немного плотнее при наведении */
  border-color: rgba(255, 255, 255, 0.9);
}

.fullscreen-image-modal .nav-button:active {
  background-color: rgba(255, 255, 255, 0.15); /* чуть темнее при нажатии */
  border-color: rgba(255, 255, 255, 0.7);
}


/* Расположение стрелок */
.fullscreen-image-modal .prev {
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
}

.fullscreen-image-modal .next {
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
}

/* Кнопка закрытия */
.fullscreen-image-modal .close-button {
  position: absolute;
  top: 30px; /* было 10px, опустили ниже */
  right: 20px;
  font-size: 26px; /* чуть меньше, чтобы выглядело аккуратнее */
  background-color: transparent; /* убрали фон полностью */
  color: white; /* лучше смотрится на затемнённом фоне */
  padding: 8px;
  border-radius: 50%;
  border: 1px solid white; /* тонкая белая рамка вместо фона */
  cursor: pointer;
  z-index: 10000;
  transition: background-color 0.2s ease;
}

.fullscreen-image-modal .close-button:hover {
  background-color: rgba(255, 255, 255, 0.2); /* лёгкое затемнение при наведении */
}

.favorite-button-top {
  position: absolute;
  top: 20px;
  right: 20px;
  font-size: 24px;
  background: transparent;
  border: none;
  cursor: pointer;
  z-index: 10;
}

.toggle-filters-button{
  background-color: #cb8ae7d2; /* темнее сиреневый при наведении */
  color: #ffffff;
  height: 50px;
  width: 200px;
  transition: all 1s;
}
  .toggle-filters-button:hover{
    background-color: #9c5fb6d1; /* темнее сиреневый при наведении */
    height: 50px;
width: 200px;}

.CENTER{
  width: 95%;
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
  
}

.knopka{
  width: 95%;
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
  margin-top: 10px;
  margin-right:  100px;
  
}

.search-container {
  width: 95%;
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
  margin-top: 10px;
  margin-right:  90px;
}

.search-input {
  margin-right:  100px;
  margin: 10px;
  height: 30px;
  font-size: 16px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 20px;
  width: 350px;
  max-width: 100%;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  transition: border-color 0.3s, box-shadow 0.3s;
}

.search-input:focus {
  border-color: #9b59b6;
  outline: none;
  box-shadow: 0 0 8px rgba(155, 89, 182, 0.3);
}

.zagolovok {
  text-align: left;
  font-size: 20px;
  font-weight: 700;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #4b0082; /* насыщенный фиолетовый */
  margin-bottom: 20px;
  text-transform: uppercase; /* делает текст полностью прописным */
  letter-spacing: 2px;
  margin-left: 100px;
}

.zagolovok1{
  justify-content: flex-start;
}

/* Обертка для фильтров и каталога */
.content-wrapper {
  display: flex;
  width: 100%;
  gap: 20px;
  justify-content: center;
  flex-wrap: nowrap;
  transition: all 0.3s ease;
}

/* Каталог товаров занимает всё пространство */
.content-wrapper .product-list {
  flex-grow: 1;
}

/* Когда фильтры открыты, сдвигаем каталог */
.content-wrapper.filters-open .filters {
  
  display: block;
  min-width: 300px;
}

.content-wrapper.filters-open .product-list {
  max-width: calc(100% - 320px); /* Учёт ширины фильтра + отступ */
}

/* Когда фильтры скрыты */
.filters {
  display: none;
}

/* Основной заголовок, центрированный, с красивым шрифтом */
.zagolovok2 {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 32px;
  font-family: 'Playfair Display', serif;
  color: #4b0082;
  font-weight: 700;
  text-align: center;
  margin-bottom: 30px;
  text-shadow: 1px 1px 2px rgba(75, 0, 130, 0.2);
}
/* Подзаголовок */
.zagolovok2 {
  margin-bottom: 25px;

}

.zagolovok3 {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 30px;
  font-weight: 800;
  color: #4b0082;
  text-transform: uppercase;
  letter-spacing: 2px;
  margin: 0 auto;
  width: 100%;
  justify-content: center;
}

.cart-button {
  background-color: #7e57c2; /* Сиреневый фон */
  color: #fff; /* Белый текст */
  font-size: 16px;
  font-weight: 600;
  padding: 12px 24px;
  margin: 10px;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease, box-shadow 0.3s ease;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  box-shadow: 0 4px 15px rgba(126, 87, 194, 0.3);
}

.cart-button:hover {
  background-color: #5e35b1; /* Более тёмный сиреневый */
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(126, 87, 194, 0.4);
}

.cart-button:active {
  transform: translateY(1px);
  box-shadow: 0 3px 10px rgba(126, 87, 194, 0.3);
}

</style>