<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click.self="closeModal">
      <div class="modal-window" role="dialog" aria-modal="true" aria-labelledby="feedback-title">
        <button class="modal-close" aria-label="Закрыть" @click="closeModal">×</button>

        <h2 id="feedback-title" class="modal-title">Оцените мероприятие</h2>

        <form @submit.prevent="submitFeedback" novalidate>
          <div class="form-group">
            <label class="form-label">Оценка мероприятия <span class="required">*</span></label>
            <div class="stars">
              <button
                v-for="star in 5"
                :key="star"
                type="button"
                class="star"
                :class="{ active: star <= rating }"
                @click="setRating(star)"
                :aria-label="`Оценка ${star} из 5`"
              >
                ★
              </button>
            </div>
            <p v-if="showError && !rating" class="error-text">
              Пожалуйста, выберите оценку
            </p>
          </div>

          <div class="form-group">
            <label class="form-label" for="feedback-comment">Комментарий</label>
            <textarea
              id="feedback-comment"
              v-model="comment"
              class="form-textarea"
              :class="{ error: commentError }"
              rows="4"
              maxlength="500"
              placeholder="Что понравилось? Что можно улучшить?"
            ></textarea>
            <div class="char-counter" :class="{ warning: comment.length > 450 }">
              {{ comment.length }} / 500
            </div>
            <p v-if="commentError" class="error-text">
              Комментарий не должен превышать 500 символов
            </p>
          </div>

          <button type="submit" class="submit-btn" :disabled="isSubmitting">
            {{ isSubmitting ? 'Отправка...' : 'Отправить отзыв' }}
          </button>
        </form>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  isOpen: { type: Boolean, default: false },
  applicationId: { type: [Number, String], default: null },
})

const emit = defineEmits(['close', 'submitted'])

const rating = ref(0)
const comment = ref('')
const showError = ref(false)
const commentError = ref(false)
const isSubmitting = ref(false)

function setRating(value) {
  rating.value = value
  showError.value = false
}

function resetForm() {
  rating.value = 0
  comment.value = ''
  showError.value = false
  commentError.value = false
}

function closeModal() {
  resetForm()
  emit('close')
}

async function submitFeedback() {
  showError.value = !rating.value
  commentError.value = comment.value.length > 500
  if (showError.value || commentError.value) return

  isSubmitting.value = true
  try {
    const payload = {
      applicationId: props.applicationId,
      rating: rating.value,
      comment: comment.value.trim(),
      createdAt: new Date().toISOString(),
    }

    try {
      await fetch('/api/feedback', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      })
    } catch (apiError) {
      const saved = JSON.parse(localStorage.getItem('feedbacks') || '[]')
      saved.push(payload)
      localStorage.setItem('feedbacks', JSON.stringify(saved))
    }

    emit('submitted', payload)
    resetForm()
  } finally {
    isSubmitting.value = false
  }
}

function onKeydown(e) {
  if (e.key === 'Escape' && props.isOpen) closeModal()
}

onMounted(() => document.addEventListener('keydown', onKeydown))
onUnmounted(() => document.removeEventListener('keydown', onKeydown))

watch(() => props.isOpen, (val) => {
  if (val) resetForm()
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.55);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 16px;
}

.modal-window {
  background: #fff;
  border-radius: 12px;
  width: 100%;
  max-width: 480px;
  max-height: 90vh;
  overflow-y: auto;
  padding: 24px;
  position: relative;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

.modal-close {
  position: absolute;
  top: 8px;
  right: 12px;
  background: none;
  border: none;
  font-size: 28px;
  cursor: pointer;
  color: #666;
  line-height: 1;
}

.modal-close:hover { color: #000; }

.modal-title {
  margin: 0 0 20px;
  font-size: 22px;
  text-align: center;
}

.form-group { margin-bottom: 18px; }

.form-label {
  display: block;
  font-weight: 600;
  margin-bottom: 8px;
  font-size: 15px;
}

.required { color: #d32f2f; }

.stars { display: flex; gap: 4px; }

.star {
  background: none;
  border: none;
  font-size: 36px;
  cursor: pointer;
  color: #ddd;
  padding: 0;
  transition: color 0.15s;
}

.star.active { color: #ffb400; }

.form-textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: inherit;
  font-size: 15px;
  resize: vertical;
  box-sizing: border-box;
}

.form-textarea.error { border-color: #d32f2f; }

.char-counter {
  text-align: right;
  font-size: 12px;
  color: #888;
  margin-top: 4px;
}

.char-counter.warning { color: #f57c00; }

.error-text {
  color: #d32f2f;
  font-size: 13px;
  margin: 6px 0 0;
}

.submit-btn {
  width: 100%;
  padding: 14px;
  background: #1976d2;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
}

.submit-btn:disabled { background: #90caf9; cursor: not-allowed; }

@media (max-width: 480px) {
  .modal-overlay { padding: 0; align-items: stretch; }
  .modal-window {
    max-width: 100%;
    max-height: 100vh;
    border-radius: 0;
    padding: 20px 16px;
  }
  .star { font-size: 44px; }
  .submit-btn { padding: 16px; font-size: 17px; }
  .form-textarea { font-size: 16px; }
}
</style>