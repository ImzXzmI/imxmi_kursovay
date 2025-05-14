<script setup>
const props = defineProps({
  favorites: Array,
});

const emit = defineEmits(["toggleFavorite", "addToCart", "openCatalog"]);

function removeFromFavorites(product) {
  emit("toggleFavorite", product);
}

function addProductToCart(product) {
  emit("addToCart", product);
  emit("toggleFavorite", product);
}


</script>

<template>
  <div class="favorites-page">
    <h1>Избранное</h1>

    <div v-if="favorites.length > 0">
      <ul class="favorites-list">
        <li v-for="(product, index) in favorites" :key="index" class="favorites-item">
          <img :src="product.images[0]" :alt="product.name" class="favorites-image" />
          <div class="favorites-info">
            <p class="favorites-name">{{ product.name }}</p>
            <p class="favorites-price">Цена: {{ product.price }} ₽</p>
            <button @click="removeFromFavorites(product)" class="remove-favorite-btn">Удалить из избранного</button>
            <button @click="addProductToCart(product)" class="add-to-cart-btn">Добавить в корзину</button>
          </div>
        </li>
      </ul>
    </div>

    <div v-else>
      <p>Ваши избранные товары пусты.</p>
    </div>

    <button class="catalog-button" @click="$emit('openCatalog')">Перейти в каталог</button>
  </div>
</template>

<style scoped>
.favorites-page{
  margin-top: 120px;
}
.favorites-list {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  justify-content: center;
  padding: 0;
  list-style: none;

}

.favorites-item {
  background: linear-gradient(135deg, #e1f5fe, #f3e5f5);
  border: 1px solid #c5cae9;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  padding: 16px;
  width: 240px;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: transform 0.3s;
}

.favorites-item:hover {
  transform: scale(1.03);
}

.favorites-image {
  width: 100%;
  height: 140px;
  object-fit: cover;
  border-radius: 12px;
  margin-bottom: 10px;
}

.favorites-info {
  text-align: center;
}

.favorites-name {
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 6px;
  color: #4a148c;
}

.favorites-price {
  font-size: 14px;
  color: #4e4e4e;
  margin: 4px 0;
}

.remove-favorite-btn, .add-to-cart-btn {
  margin-top: 8px;
  padding: 8px 16px;
  background-color: #ff4081;
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: background 0.3s;
}

.remove-favorite-btn:hover {
  background-color: #f50057;
}

.add-to-cart-btn {
  background-color: #D6A9FD;
  color: #000000;
}

.add-to-cart-btn:hover {
  background-color: #a779cf;
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
