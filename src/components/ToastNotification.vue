<template>
  <Transition name="toast">
    <div v-if="visible" class="toast" role="status">
      {{ message }}
    </div>
  </Transition>
</template>

<script setup>
defineProps({
  visible: { type: Boolean, default: false },
  message: { type: String, default: 'Спасибо за отзыв!' },
})
</script>

<style scoped>
.toast {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: #2e7d32;
  color: #fff;
  padding: 14px 24px;
  border-radius: 8px;
  font-size: 15px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
  z-index: 2000;
  max-width: 90vw;
}

.toast-enter-active, .toast-leave-active { transition: opacity 0.3s, transform 0.3s; }
.toast-enter-from, .toast-leave-to { opacity: 0; transform: translate(-50%, -20px); }
</style>