<template>
  <div>
    <header class="app-header">
      <div class="container">
        <h1 class="app-title">Моя библиотека</h1>
        <div class="app-subtitle">Личная коллекция книг</div>
      </div>
    </header>

    <div class="stats-bar">
      <div class="container d-flex align-items-center flex-wrap gap-2">
        <div class="stat-item">
          <span class="stat-number">{{ books.length }}</span>
          <span class="stat-label">всего</span>
        </div>
        <div class="stat-divider"></div>
        <div class="stat-item">
          <span class="stat-number">{{ readCount }}</span>
          <span class="stat-label">прочитано</span>
        </div>
        <div class="stat-divider"></div>
        <div class="stat-item">
          <span class="stat-number">{{ books.length - readCount }}</span>
          <span class="stat-label">в очереди</span>
        </div>
        <div class="stat-divider d-none d-sm-block"></div>
        <div class="stat-item d-none d-sm-flex">
          <span class="stat-number">{{ avgRating }}</span>
          <span class="stat-label">средняя оценка</span>
        </div>

        <div class="ms-auto">
          <div class="search-wrap">
            <span class="search-icon">🔍</span>
            <input
              v-model="searchQuery"
              class="search-input"
              placeholder="Поиск по названию или автору..."
            />
          </div>
        </div>
      </div>
    </div>

    <div class="filter-tabs">
      <div class="container d-flex">
        <button
          v-for="tab in tabs"
          :key="tab.value"
          class="filter-tab"
          :class="{ active: activeFilter === tab.value }"
          @click="activeFilter = tab.value"
        >{{ tab.label }}</button>
      </div>
    </div>

    <div class="sort-controls">
      <div class="container d-flex align-items-center gap-2 flex-wrap">
        <span class="sort-label">Сортировка:</span>
        <button
          v-for="s in sortOptions"
          :key="s.value"
          class="sort-btn"
          :class="{ active: sortBy === s.value }"
          @click="sortBy = s.value"
        >{{ s.label }}</button>
      </div>
    </div>

    <main class="container py-4">
      <div v-if="filteredBooks.length === 0" class="empty-state">
        <div class="empty-icon">📚</div>
        <div class="empty-title">
          {{ books.length === 0 ? 'Библиотека пуста' : 'Ничего не найдено' }}
        </div>
        <div class="empty-text">
          {{ books.length === 0
            ? 'Нажмите + чтобы добавить первую книгу'
            : 'Попробуйте изменить фильтр или поисковый запрос' }}
        </div>
      </div>

      <div v-else class="row g-3">
        <div
          v-for="book in filteredBooks"
          :key="book.id"
          class="col-6 col-md-4 col-lg-3"
        >
          <BookCard
            :book="book"
            @toggle-read="toggleRead"
            @delete="deleteBook"
            @update-rating="updateRating"
            @update-cover="updateCover"
          />
        </div>
      </div>
    </main>

    <button class="fab-add" @click="showModal = true" title="Добавить книгу">+</button>

    <AddBookModal
      v-if="showModal"
      @close="showModal = false"
      @add="addBook"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import BookCard from './components/BookCard.vue'
import AddBookModal from './components/AddBookModal.vue'

const showModal = ref(false)
const activeFilter = ref('all')
const sortBy = ref('date')
const searchQuery = ref('')

const tabs = [
  { label: 'Все', value: 'all' },
  { label: 'Прочитано', value: 'read' },
  { label: 'К прочтению', value: 'unread' },
  { label: 'С оценкой', value: 'rated' },
]

const sortOptions = [
  { label: 'Дата добавления', value: 'date' },
  { label: 'Название', value: 'title' },
  { label: 'Автор', value: 'author' },
  { label: 'Оценка', value: 'rating' },
]

const books = ref([
  {
    id: 1,
    title: 'Мастер и Маргарита',
    author: 'Михаил Булгаков',
    genre: 'Классика',
    description: 'Роман о визите дьявола в советскую Москву, переплетённый с историей Понтия Пилата и Иешуа.',
    rating: 5,
    read: true,
    cover: null,
    addedAt: Date.now() - 86400000 * 3,
  },
  {
    id: 2,
    title: 'Преступление и наказание',
    author: 'Фёдор Достоевский',
    genre: 'Классика',
    description: 'История студента Раскольникова, решившего проверить собственную теорию и совершившего убийство.',
    rating: 4,
    read: true,
    cover: null,
    addedAt: Date.now() - 86400000 * 10,
  },
  {
    id: 3,
    title: 'Сто лет одиночества',
    author: 'Габриэль Гарсиа Маркес',
    genre: 'Роман',
    description: 'Магический реализм о судьбах семьи Буэндиа в вымышленном городке Макондо.',
    rating: 0,
    read: false,
    cover: null,
    addedAt: Date.now() - 86400000 * 1,
  },
])

const readCount = computed(() => books.value.filter(b => b.read).length)
const avgRating = computed(() => {
  const rated = books.value.filter(b => b.rating > 0)
  if (!rated.length) return '—'
  return (rated.reduce((s, b) => s + b.rating, 0) / rated.length).toFixed(1)
})

const filteredBooks = computed(() => {
  let list = [...books.value]

  // Поиск
  const q = searchQuery.value.toLowerCase().trim()
  if (q) list = list.filter(b =>
    b.title.toLowerCase().includes(q) ||
    (b.author && b.author.toLowerCase().includes(q))
  )

  // Фильтр
  if (activeFilter.value === 'read') list = list.filter(b => b.read)
  if (activeFilter.value === 'unread') list = list.filter(b => !b.read)
  if (activeFilter.value === 'rated') list = list.filter(b => b.rating > 0)

  // Сортировка
  list.sort((a, b) => {
    if (sortBy.value === 'title') return a.title.localeCompare(b.title)
    if (sortBy.value === 'author') return (a.author || '').localeCompare(b.author || '')
    if (sortBy.value === 'rating') return b.rating - a.rating
    return b.addedAt - a.addedAt
  })

  return list
})

const addBook = (data) => {
  books.value.unshift({
    id: Date.now(),
    addedAt: Date.now(),
    read: false,
    ...data,
  })
}

const toggleRead = (id) => {
  const b = books.value.find(b => b.id === id)
  if (b) b.read = !b.read
}

const deleteBook = (id) => {
  books.value = books.value.filter(b => b.id !== id)
}

const updateRating = (id, rating) => {
  const b = books.value.find(b => b.id === id)
  if (b) b.rating = rating
}

const updateCover = (id, cover) => {
  const b = books.value.find(b => b.id === id)
  if (b) b.cover = cover
}
</script>
