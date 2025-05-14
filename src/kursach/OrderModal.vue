<script setup>
</script>

<template>
    <div class="modal-overlay" @click.self="$emit('close')">
      <div class="modal-content">
        <h2>Оформление заказа</h2>
        <div v-if="!isAuthenticated">
          <p>Пожалуйста, <strong>зарегистрируйтесь</strong> перед оформлением заказа.</p>

        </div>
        <div v-else>
          <p>Вы авторизованы. Перейдите к оформлению заказа.</p>
          <button @click="$emit('goToCheckout')">Перейти к оформлению</button>
        </div>
        <button @click="$emit('close')">Закрыть</button>
      </div>
    </div>
  </template>
  
  <script setup>
  const props = defineProps({
    isAuthenticated: Boolean,
  });
  </script>
  
  <style scoped>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 999;
  }
  
  .modal-content {
    background: white;
    padding: 20px;
    border-radius: 12px;
    text-align: center;
    min-width: 300px;
  }
  </style>
  