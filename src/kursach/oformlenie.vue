<script setup>
import { reactive, computed, ref } from 'vue'

const forma = reactive({
  firstName: '',
  lastName: '',
  middleName: '',
  phone: '',
  email: '',
  delivery: '',
  payment: '',
  address: '',
  postalCode: '',
  cardNumber: '',
  cardExpiry: '',
  cardCVV: '',
  saveCard: false,
  comment: '',
  giftWrap: false,
})

const addresses = [
  'ул. Пушкина, д. 10, кв. 5, г. Москва',
  'пр. Ленина, д. 25, г. Санкт-Петербург',
]

const emit = defineEmits(['updateForm', 'close', 'clearCart'])

const orderConfirmed = ref(false)
const submittedOrder = ref(null)

const firstNameError = computed(() => /^[А-ЯЁ][а-яё]*$/.test(forma.firstName) ? '' : 'Имя должно начинаться с заглавной буквы')
const lastNameError = computed(() => /^[А-ЯЁ][а-яё]*$/.test(forma.lastName) ? '' : 'Фамилия должна начинаться с заглавной буквы')
const phoneError = computed(() => /^\+7/.test(forma.phone) ? '' : 'Телефон должен быть в формате +7 (___) ___-__-__')
const emailError = computed(() => /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(forma.email) ? '' : 'Введите корректный email')
const postalCodeError = computed(() => /^\d{6}$/.test(forma.postalCode) ? '' : 'Индекс должен состоять из 6 цифр')

function submitOrder() {
  console.log('submitOrder called') // Добавим отладочный вывод

  // Валидация
  if (
    firstNameError.value ||
    lastNameError.value ||
    phoneError.value ||
    emailError.value ||
    !forma.delivery ||
    !forma.payment
  ) {
    console.log('Validation failed') // Сообщение об ошибке валидации
    return
  }

  if (forma.delivery === 'mail' && postalCodeError.value) {
    console.log('Postal code validation failed') // Ошибка индекса
    return
  }

  // Создание заказа
  submittedOrder.value = { ...forma }
  orderConfirmed.value = true

  console.log('Order submitted', submittedOrder.value) // Выводим заказ

  // Отправка данных заказа родителю и очищение корзины
  emit('updateForm', submittedOrder.value)
  emit('clearCart')
}
</script>

<template>
  <div class="modal-overlay">
    <div class="modal-content">
      <h2 v-if="!orderConfirmed">Оформление заказа</h2>
      <h2 v-else>Заказ успешно оформлен</h2>

      <div v-if="!orderConfirmed">
        <form class="order-form" @submit.prevent="submitOrder">
          <!-- Имя -->
          <div class="form-group">
            <label for="firstName">Имя</label>
            <input id="firstName" v-model="forma.firstName" type="text" required />
            <span v-if="firstNameError" class="error-text">{{ firstNameError }}</span>
          </div>

          <!-- Фамилия -->
          <div class="form-group">
            <label for="lastName">Фамилия</label>
            <input id="lastName" v-model="forma.lastName" type="text" required />
            <span v-if="lastNameError" class="error-text">{{ lastNameError }}</span>
          </div>

          <!-- Отчество -->
          <div class="form-group">
            <label for="middleName">Отчество</label>
            <input id="middleName" v-model="forma.middleName" type="text" />
          </div>

          <!-- Телефон -->
          <div class="form-group">
            <label for="phone">Телефон</label>
            <input id="phone" v-model="forma.phone" type="tel" required placeholder="+7 (___) ___-__-__" />
            <span v-if="phoneError" class="error-text">{{ phoneError }}</span>
          </div>

          <!-- Email -->
          <div class="form-group">
            <label for="email">Email</label>
            <input id="email" v-model="forma.email" type="email" required />
            <span v-if="emailError" class="error-text">{{ emailError }}</span>
          </div>

          <!-- Доставка -->
          <div class="form-group">
            <label for="delivery">Способ доставки</label>
            <select id="delivery" v-model="forma.delivery" required>
              <option value="" disabled>Выберите способ</option>
              <option value="КУРЬЕР">Курьер</option>
              <option value="САМОВЫВОЗ">Самовывоз</option>
              <option value="ПОЧТА">Почта</option>
            </select>
          </div>

          <div v-if="forma.delivery === 'КУРЬЕР'" class="form-group">
            <label for="address">Введите адрес</label>
            <input id="address" v-model="forma.address" type="text" required />
          </div>

          <div v-if="forma.delivery === 'САМОВЫВОЗ'" class="form-group">
            <label for="address">Выберите адрес</label>
            <select id="address" v-model="forma.address" required>
              <option value="" disabled>Выберите адрес</option>
              <option v-for="address in addresses" :key="address" :value="address">{{ address }}</option>
            </select>
          </div>

          <div v-if="forma.delivery === 'ПОЧТА'" class="form-group">
            <label for="postalCode">Индекс</label>
            <input id="postalCode" v-model="forma.postalCode" type="text" required />
            <span v-if="postalCodeError" class="error-text">{{ postalCodeError }}</span>
          </div>

          <!-- Оплата -->
          <div class="form-group">
            <label for="payment">Способ оплаты</label>
            <select id="payment" v-model="forma.payment" required>
              <option value="" disabled>Выберите способ оплаты</option>
              <option value="spb">СБП</option>
              <option value="card">По карте</option>
            </select>
          </div>

          <div v-if="forma.payment === 'card'" class="form-group">
            <label for="cardNumber">Номер карты</label>
            <input id="cardNumber" v-model="forma.cardNumber" type="text" required />
          </div>

          <div v-if="forma.payment === 'card'" class="form-group">
            <label for="cardExpiry">Срок действия</label>
            <input id="cardExpiry" v-model="forma.cardExpiry" type="text" required placeholder="ММ/ГГ" />
          </div>

          <div v-if="forma.payment === 'card'" class="form-group">
            <label for="cardCVV">CVV</label>
            <input id="cardCVV" v-model="forma.cardCVV" type="text" required />
          </div>

          <div v-if="forma.payment === 'card'" class="form-group checkbox">
            <label>
              <input type="checkbox" v-model="forma.saveCard" />
              Сохранить номер карты
            </label>
          </div>

          <div class="form-group">
            <label for="comment">Комментарий</label>
            <textarea id="comment" v-model="forma.comment" rows="2" placeholder="Дополнительная информация"></textarea>
          </div>

          <div class="form-group checkbox">
            <label>
              <input type="checkbox" v-model="forma.giftWrap" />
              Подарочная упаковка
            </label>
          </div>

          <div class="form-buttons">
            <button type="submit" class="primary">Оформить заказ</button>
            <button type="button" class="secondary" @click="emit('close')">Вернуться в корзину</button>
          </div>
        </form>
      </div>

      <!-- После оформления -->
      <div v-else class="order-confirmation">
        <p><strong>Имя:</strong> {{ submittedOrder.firstName }}</p>
        <p><strong>Фамилия:</strong> {{ submittedOrder.lastName }}</p>
        <p><strong>Телефон:</strong> {{ submittedOrder.phone }}</p>
        <p><strong>Email:</strong> {{ submittedOrder.email }}</p>
        <p><strong>Доставка:</strong> {{ submittedOrder.delivery }}</p>
        <p v-if="submittedOrder.address"><strong>Адрес:</strong> {{ submittedOrder.address }}</p>
        <p v-if="submittedOrder.postalCode"><strong>Индекс:</strong> {{ submittedOrder.postalCode }}</p>
        <p><strong>Оплата:</strong> {{ submittedOrder.payment }}</p>
        <p v-if="submittedOrder.payment === 'card'"><strong>Номер карты:</strong> {{ submittedOrder.cardNumber }}</p>
        <p v-if="submittedOrder.comment"><strong>Комментарий:</strong> {{ submittedOrder.comment }}</p>
        <p><strong>Подарочная упаковка:</strong> {{ submittedOrder.giftWrap ? 'Да' : 'Нет' }}</p>

        <button class="secondary" @click="emit('close')">Закрыть</button>
      </div>
    </div>
  </div>
</template>


<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: #fff;
  padding: 2rem;
  width: 100%;
  max-width: 600px;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
  max-height: 90vh;
  overflow-y: auto;
}

h2 {
  margin-bottom: 1.5rem;
  text-align: center;
  color: #000000;
}

.order-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-group label {
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-group input,
.form-group textarea,
.form-group select {
  padding: 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  font-size: 1rem;
  resize: vertical;
}

.form-group.checkbox {
  flex-direction: row;
  align-items: center;
}

.form-group.checkbox label {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 0;
  font-weight: 400;
}

.form-buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 1.5rem;
}

.form-buttons button {
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s ease;
}

button.primary {
  background-color: #9AE3F3;
  color: #000000;
  border: none;
}

button.primary:hover {
  background-color: #5eaabb;
}

button.secondary {
  background-color: #e5e7eb;
  color: #333;
  border: none;
}

button.secondary:hover {
  background-color: #d1d5db;
}

.error-text {
  color: red;
  font-size: 0.9rem;
  margin-top: 0.25rem;
}
</style>
