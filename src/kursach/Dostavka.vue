<script setup>
import { ref } from 'vue';

const deliverySteps = [
  { id: 1, title: 'Размещение заказа', description: 'Выберите украшение и оформите заказ онлайн.' },
  { id: 2, title: 'Упаковка', description: 'Каждое украшение аккуратно упаковывается для защиты и презентабельности.' },
  { id: 3, title: 'Транспортировка', description: 'Мы используем надежных курьеров для доставки по России.' },
  { id: 4, title: 'Доставка', description: 'Доставка осуществляется Почтой России, СДЕК или курьером от двери до двери.' },
  { id: 5, title: 'Бесплатная доставка', description: 'Доставка бесплатна при сумме заказа от 8000 рублей.' },
];
</script>

<template>
  <div class="delivery-process">
    <h1>Процесс доставки украшений ручной работы</h1>

    <div class="steps">
      <div v-for="step in deliverySteps" :key="step.id" class="step">
        <h2>{{ step.title }}</h2>
        <p>{{ step.description }}</p>
        <div class="arrow-animation"></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.delivery-process {
  max-width: 900px;
  margin: 0 auto;
  padding: 40px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(to bottom, #ffffff, #e0f7fa 50%, #d1c4e9 100%);
  margin-top: 60px;
}

h1 {
  text-align: center;
  color: #4a148c; /* Темный фиолетовый */
  font-size: 32px;
  margin-bottom: 20px;
  font-weight: 600;
}

.steps {
  margin-top: 30px;
}

.step {
  background: #ffffff;
  border-radius: 15px;
  padding: 25px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  margin-bottom: 25px;
  transition: transform 0.3s ease, background 0.3s ease;
}

.step:hover {
  transform: scale(1.05);
  background: rgba(126, 87, 194, 0.1); /* Фиолетовый с прозрачностью */
  box-shadow: 0 4px 25px rgba(0, 0, 0, 0.15);
}

h2 {
  color: #7e57c2; /* Сиреневый цвет */
  font-size: 24px;
  margin-bottom: 10px;
}

p {
  color: #555;
  font-size: 16px;
  line-height: 1.6;
}

.arrow-animation {
  margin-top: 15px;
  width: 50px;
  height: 10px;
  background: #7e57c2; /* Сиреневый для стрелки */
  animation: moveArrow 2s infinite;
  border-radius: 5px;
}

@keyframes moveArrow {
  0% {
    transform: translateX(0);
  }
  50% {
    transform: translateX(10px);
  }
  100% {
    transform: translateX(0);
  }
}

.cta-button-container {
  text-align: center;
  margin-top: 40px;
}

.cta-button {
  padding: 12px 30px;
  font-size: 18px;
  background-color: #7e57c2; /* Сиреневый цвет кнопки */
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.cta-button:hover {
  background-color: #5e35b1; /* Темный сиреневый при наведении */
  transform: translateY(-3px);
}

.cta-button:active {
  transform: translateY(1px);
}
</style>
