<script setup>
import { ref, onMounted } from "vue";

const props = defineProps({
  products: Array,
  isFavorite: Function,
  cart: Array
});

// При монтировании компонента прокручиваем страницу вверх
onMounted(() => {
  window.scrollTo({ top: 0, behavior: 'smooth' }); // Плавная прокрутка вверх
});

const currentSlide = ref(0);

const slides = [
  "https://imzxzmi.github.io/imxmi_kursovay/img/j1.1.jpg",
  "https://imzxzmi.github.io/imxmi_kursovay/img/j2.1.jpg",
  "https://imzxzmi.github.io/imxmi_kursovay/img/j3.1.jpg",
];

const nextSlide = () => {
  currentSlide.value = (currentSlide.value + 1) % slides.length;
};

const prevSlide = () => {
  currentSlide.value = (currentSlide.value - 1 + slides.length) % slides.length;
};

// Автоматическая смена слайдов (по желанию)
onMounted(() => {
  setInterval(nextSlide, 5000);
});
</script>

<template>
  <div class="home-page">
    <!-- Hero-секция с фоном и CTA -->
    <section class="hero">
      <div class="hero-content">
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/logo.png" alt="Логотип" class="hero-logo" />
        <h1>Украшения ручной работы</h1>
        <p>Созданные с любовью. Носимые с гордостью.</p>
        <button class="catalog-button" @click="$emit('openCatalog')">Перейти в каталог</button>
      </div>
    </section>

            <!-- Галерея фото -->
    <div class="photo-gallery">
      <div class="photo-row">
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore.jpg" alt="Фото 1" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore1.jpg" alt="Фото 2" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore3.jpg" alt="Фото 3" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore4.jpg" alt="Фото 4" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore9.jpg" alt="Фото 4" />
      </div>
    </div>

    <section class="text-wrapper">
      <div class="text">
        Кольчужные украшения — это изделия, созданные в технике кольчужного плетения, то есть 
        путём переплетения между собой металлических колец.
      </div>  
      <div class="text">
        Castlecore — это эстетика, вдохновлённая средневековыми замками, королевскими 
        дворами и рыцарской культурой. Она сочетает в себе элегантность прошлого с современными интерпретациями.  
      </div>
    </section>

        <!-- Галерея фото -->
        <div class="photo-gallery">
      <div class="photo-row">
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore10.jpg" alt="Фото 4" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore7.jpg" alt="Фото 7" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore8.jpg" alt="Фото 8" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore6.jpg" alt="Фото 7" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/castlcore5.jpg" alt="Фото 8" />
      </div>
    </div>

    <section class="text-wrapper">
      <div class="text">
        Процесс кольчужного плетения требует высокой точности и терпения. Каждое кольцо проходит через
         несколько других колец, и это плетение может продолжаться в бесконечном цикле. Создание кольчуги 
         или кольчужного украшения может занять недели или даже месяцы, в зависимости от сложности дизайна. 
      </div>
    </section>

    <section class="custom-slider">
    <div class="slider-frame">
      <img :src="slides[currentSlide]" alt="Слайд" class="slider-image" />

      <button class="slider-btn prev" @click="prevSlide">❮</button>
      <button class="slider-btn next" @click="nextSlide">❯</button>

      <div class="dots">
        <span
          v-for="(slide, index) in slides"
          :key="index"
          class="dot"
          :class="{ active: index === currentSlide }"
          @click="currentSlide = index"
        ></span>
      </div>
    </div>
  </section>

  <section class="text-wrapper">
      <div class="text">
        Сегодня кольчужное плетение активно используется не только для доспехов, но и в ювелирных изделиях. 
        Кольчужные браслеты, ожерелья и кольца становятся популярными благодаря своей уникальной структуре и эффектному виду. 
        Такие украшения могут быть выполнены из различных материалов: от обычного металла до золота, серебра и даже титановых сплавов.
      </div>
    </section>

    <!-- Преимущества -->
    <section class="features">
      <div class="feature">
        <h3>Уникальный дизайн</h3>
        <p>Каждое украшение – эксклюзивный шедевр ручной работы.</p>
      </div>
      <div class="feature">
        <h3>Премиальные материалы</h3>
        <p>Используем только проверенные и безопасные материалы.</p>
      </div>
      <div class="feature">
        <h3>Быстрая доставка</h3>
        <p>Оперативная отправка по всей России и за её пределы.</p>
      </div>
    </section>

    <section class="text-wrapper">
      <div class="text">
        В некоторых культурах кольчужные украшения носили как амулеты или символы силы и 
        мужества. Например, викинги часто использовали кольчужные ожерелья и браслеты как 
        защитные символы, веря, что такие изделия могут защитить их от зла.
      </div>
      <div class="text">
        Современные мастера и дизайнеры используют кольчужное плетение не только в традиционном, 
        но и в новых формах. Некоторые используют кольца разных размеров или создают цветные 
        кольчужные украшения, комбинируя металл с камнями или стеклом. Это делает изделия более 
        креативными и подходящими для повседневного ношения.
      </div>
      <div class="text">
        Техника кольчужного плетения является не только предметом исторических исследований, но 
        и активно используется современными реконструкторами, которые воссоздают средневековые 
        доспехи для исторических фестивалей и ролевых игр.
      </div>
    </section>

    <div class="photo-gallery">
      <div class="photo-row">
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/j5.1.jpg" alt="Фото 4" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/j1.1.jpg" alt="Фото 7" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/j5.3.jpg" alt="Фото 8" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/j3.3.jpg" alt="Фото 7" />
        <img src="https://imzxzmi.github.io/imxmi_kursovay/img/j5.2.jpg" alt="Фото 8" />
      </div>
    </div>




    <!-- Почему выбирают нас -->
    <section class="why-us">
      <h2>Почему выбирают нас</h2>
      <p>Мы объединяем древние техники с современным стилем, чтобы создать украшения, которые подчеркивают индивидуальность.</p>
    </section>
  </div>

  
</template>

<style scoped>
.products-carousel {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
}

.carousel-btn {
  background-color: #f0f0f0;
  border: none;
  padding: 10px;
  cursor: pointer;
  font-size: 18px;
}

.products-wrapper {
  display: flex;
  overflow: hidden;
}

.product-card {
  width: 200px;
  margin: 10px;
  text-align: center;
}

.product-image {
  width: 100%;
  height: auto;
}

.product-info {
  padding: 10px;
}

button {
  color: white;
  border: none;
  padding: 10px;
  cursor: pointer;
}

.cart {
  margin-top: 20px;
}
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body, html {
  width: 100%;
  height: 100%;
  margin: 0;
}

.photo-gallery {
  margin: 0 auto;
  padding: 20px 10px;
  max-width: 1200px;
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.photo-row {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 20px;
}

.photo-row img {
  width: calc(20% - 16px);
  min-width: 180px;
  height: auto;
  border-radius: 12px;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  object-fit: cover;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.photo-row img:hover {
  transform: scale(1.05);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
}


.home-page {
  margin: 0; /* Убираем отступы по бокам и сверху */
  padding: 0; /* Убираем внутренние отступы */
}

/* Hero-секция */
.hero {
  background: url('https://imzxzmi.github.io/imxmi_kursovay/img/fon.jpg') center/cover no-repeat;
  height: 110vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: #9AE3F3;
  position: relative;
  margin: 0; /* Убираем любые отступы */
  padding: 0; /* Убираем любые отступы */
  top: 0; /* Убираем отступ сверху */
}

.hero::before {
  content: "";
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.55);
}

.hero-content {
  position: relative;
  z-index: 1;
  max-width: 800px;
}

.hero-logo {
  width: 500px;
  margin-bottom: 20px;
  filter: drop-shadow(0 5px 10px rgba(0, 0, 0, 0.7));
}

.hero h1 {
  font-size: 3rem;
  margin-bottom: 10px;
}

.hero p {
  font-size: 1.25rem;
  margin-bottom: 20px;
}

.catalog-button {
  background-color: #D6A9FD;
  border-radius: 25px;
  color: white;
  padding: 12px 25px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: 0.3s;
}

.catalog-button:hover {
  background-color: #a77fc9;
}

/* Слайдер */
.slider {
  margin: 50px auto;
  position: relative;
  width: 90%;
  max-width: 1200px;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2);
}

.slider-container {
  position: relative;
}

.slider-image {
  width: 100%;
  height: 500px;
  object-fit: cover;
}

.slider-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 2rem;
  color: white;
  background: rgba(0, 0, 0, 0.4);
  padding: 10px;
  cursor: pointer;
  z-index: 2;
}

.prev {
  left: 10px;
}

.next {
  right: 10px;
}


/* Преимущества */
.features {
  display: flex;
  justify-content: space-around;
  padding: 40px 20px;
  background-color: rgba(255, 255, 255, 0.658);
  text-align: center;
}

.feature {
  max-width: 300px;
}

.feature img {
  width: 60px;
  margin-bottom: 15px;
}

.feature h3 {
  margin-bottom: 10px;
}

/* Процесс */
.process {
  padding: 60px 20px;
}

.process h2 {
  text-align: center;
  margin-bottom: 40px;
}

.process-step {
  margin-bottom: 30px;
  text-align: center;
}

.process-step img {
  width: 100%;
  max-width: 700px;
  border-radius: 10px;
}

/* Почему выбирают нас */
.why-us {
  padding: 60px 20px;
  text-align: center;
}

.why-us h2 {
  margin-bottom: 20px;
}

.why-us p {
  max-width: 800px;
  margin: 0 auto;
  font-size: 1.1rem;
  line-height: 1.6;
}

@media (max-width: 768px) {
  .photo-row img {
    max-width: 48%;
  }

  .slider-container {
    height: 350px;
  }

  .hero-logo {
    width: 70%;
  }

  .catalog-button {
    width: 100%;
    padding: 15px;
  }
}


.text-wrapper {
  display: flex;
  justify-content: center;
  padding: 40px 20px;
}

.text {
  background: rgba(255, 255, 255, 0.7); /* Полупрозрачный белый */
  width: 70%;
  padding: 30px;
  margin: 10px;
  border-radius: 20px;
  font-size: 18px;
  line-height: 1.6;
  color: #333;
  font-family: 'Georgia', serif;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
  text-align: center;
}

.custom-slider {
  display: flex;
  justify-content: center;
  padding: 40px 20px;

}

.slider-frame {
  position: relative;
  width: 100%;
  max-width: 600px;
  aspect-ratio: 1 / 1; /* Квадрат */
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
}

.slider-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: opacity 0.6s ease-in-out;
  border-radius: 20px;
}

.slider-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 32px;
  cursor: pointer;
  padding: 8px 12px;
  border-radius: 50%;
  transition: background 0.3s ease;
  z-index: 2;
}


.prev {
  left: 15px;
}

.next {
  right: 15px;
}

.dots {
  display: flex;
  justify-content: center;
  padding: 12px;
  position: absolute;
  bottom: 10px;
  width: 100%;
  gap: 8px;
}

.dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: background 0.3s ease;
}

.dot.active {
  background: #7e57c2;
}

.slider-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 32px;
  cursor: pointer;
  padding: 8px 12px;
  background: none; /* убираем фон */
  border: none; /* убираем рамку */
  color: white; /* исходный цвет */
  transition: color 0.3s ease;
  z-index: 2;
}

.slider-btn:hover {
  color: #7e57c2; /* сиреневый при наведении */
}

</style>
