<template>
  <div class="book-card" :class="{ read: book.read }">

    <div class="book-cover-wrap" @click="triggerCoverUpload">
      <img v-if="book.cover" :src="book.cover" :alt="book.title" class="book-cover-img" />
      <div v-else class="book-cover-placeholder">
        <span class="cover-icon">📖</span>
        <span class="cover-upload-hint">Добавить обложку</span>
      </div>
      <span v-if="book.read" class="read-badge">Прочитано</span>
      <input
        ref="coverInput"
        type="file"
        accept="image/*"
        class="cover-input"
        @change="onCoverUpload"
      />
    </div>

    <div class="book-body">
      <div v-if="book.genre" class="book-genre">{{ book.genre }}</div>
      <div class="book-title">{{ book.title }}</div>
      <div class="book-author">{{ book.author || 'Автор не указан' }}</div>
      <div v-if="book.description" class="book-description">{{ book.description }}</div>

      <div class="star-rating">
        <span
          v-for="i in 5"
          :key="i"
          class="star"
          :class="{ filled: i <= (hoverRating || book.rating) }"
          @mouseenter="hoverRating = i"
          @mouseleave="hoverRating = 0"
          @click="setRating(i)"
        >★</span>
      </div>

      <div class="card-actions">
        <button
          class="btn-read-toggle"
          :class="{ read: book.read }"
          @click="$emit('toggle-read', book.id)"
        >
          {{ book.read ? '✓ Прочитано' : 'Отметить' }}
        </button>
        <button class="btn-delete" @click="$emit('delete', book.id)" title="Удалить">✕</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({ book: Object })
const emit = defineEmits(['toggle-read', 'delete', 'update-rating', 'update-cover'])

const coverInput = ref(null)
const hoverRating = ref(0)

const triggerCoverUpload = () => coverInput.value?.click()

const onCoverUpload = (e) => {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (ev) => emit('update-cover', props.book.id, ev.target.result)
  reader.readAsDataURL(file)
}

const setRating = (val) => {
  emit('update-rating', props.book.id, val === props.book.rating ? 0 : val)
}
</script>
