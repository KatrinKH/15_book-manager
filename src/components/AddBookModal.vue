<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-box">
      <div class="modal-header">
        <h2 class="modal-title">Добавить книгу</h2>
        <button class="modal-close" @click="$emit('close')">✕</button>
      </div>
      <div class="modal-body">

        <label class="cover-upload-area" @click="triggerCover">
          <img v-if="form.cover" :src="form.cover" alt="cover" />
          <template v-else>
            <div class="cover-upload-icon">🖼</div>
            <div class="cover-upload-label">Нажмите чтобы загрузить обложку<br><small style="opacity:.6">(необязательно)</small></div>
          </template>
          <input ref="coverRef" type="file" accept="image/*" style="display:none" @change="onCover" />
        </label>

        <div class="row g-3">
          <div class="col-12">
            <label class="form-label-custom">Название *</label>
            <input v-model="form.title" class="form-control-custom" placeholder="Введите название" required />
          </div>
          <div class="col-12">
            <label class="form-label-custom">Автор</label>
            <input v-model="form.author" class="form-control-custom" placeholder="Имя автора" />
          </div>
          <div class="col-12">
            <label class="form-label-custom">Жанр</label>
            <select v-model="form.genre" class="form-control-custom form-select-custom">
              <option value="">— Выберите жанр —</option>
              <option v-for="g in genres" :key="g">{{ g }}</option>
            </select>
          </div>
          <div class="col-12">
            <label class="form-label-custom">Описание</label>
            <textarea v-model="form.description" class="form-control-custom" placeholder="Краткое описание книги..." rows="3"></textarea>
          </div>
          <div class="col-12">
            <label class="form-label-custom">Оценка</label>
            <div class="star-rating">
              <span
                v-for="i in 5" :key="i"
                class="star"
                :class="{ filled: i <= (hoverStar || form.rating) }"
                style="font-size:1.3rem"
                @mouseenter="hoverStar = i"
                @mouseleave="hoverStar = 0"
                @click="form.rating = form.rating === i ? 0 : i"
              >★</span>
            </div>
          </div>
        </div>

        <button class="btn-submit" :disabled="!form.title.trim()" @click="submit">
          Добавить в библиотеку
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const emit = defineEmits(['close', 'add'])

const coverRef = ref(null)
const hoverStar = ref(0)

const genres = [
  'Классика', 'Роман', 'Фантастика', 'Фэнтези', 'Детектив',
  'Биография', 'История', 'Психология', 'Философия', 'Поэзия', 'Другое'
]

const form = reactive({
  title: '',
  author: '',
  genre: '',
  description: '',
  rating: 0,
  cover: null,
})

const triggerCover = () => coverRef.value?.click()

const onCover = (e) => {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (ev) => { form.cover = ev.target.result }
  reader.readAsDataURL(file)
}

const submit = () => {
  if (!form.title.trim()) return
  emit('add', { ...form })
  emit('close')
}
</script>
