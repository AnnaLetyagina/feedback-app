<template>
  <div class="applications-page">
    <h1>Мои заявки</h1>

    <!-- ... список заявок ... -->

    <div v-if="eventFinished" class="feedback-block">
      <p>Мероприятие завершено. Поделитесь впечатлениями!</p>

      <button
        v-if="!feedbackSent"
        class="open-feedback-btn"
        @click="isModalOpen = true"
      >
        Оставить отзыв
      </button>

      <p v-if="feedbackSent" class="feedback-sent-text">Ваш отзыв отправлен</p>
    </div>

    <FeedbackModal
      :is-open="isModalOpen"
      :application-id="currentApplicationId"
      @close="isModalOpen = false"
      @submitted="onFeedbackSubmitted"
    />

    <ToastNotification :visible="toastVisible" message="Спасибо за отзыв!" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import FeedbackModal from './FeedbackModal.vue'
import ToastNotification from './ToastNotification.vue'

const isModalOpen = ref(false)
const toastVisible = ref(false)
const feedbackSent = ref(false)
const eventFinished = ref(true)
const currentApplicationId = ref(1)

function onFeedbackSubmitted(payload) {
  console.log('Отзыв отправлен:', payload)
  isModalOpen.value = false
  feedbackSent.value = true
  toastVisible.value = true
  setTimeout(() => { toastVisible.value = false }, 3000)
}
</script>

<style scoped>
.feedback-block { margin-top: 24px; text-align: center; }
.open-feedback-btn {
  padding: 12px 24px;
  background: #1976d2;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
}
.open-feedback-btn:hover { background: #1565c0; }
.feedback-sent-text { color: #2e7d32; font-weight: 600; margin-top: 12px; }
</style>