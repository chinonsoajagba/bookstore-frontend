<script setup>
const { lessons, searchQuery, sortBy, sortOrder, showCart, isLoading } = defineProps({
  lessons: {
    type: Array,
    required: true,
  },
  searchQuery: {
    type: String,
    required: true,
  },
  sortBy: {
    type: String,
    required: true,
  },
  sortOrder: {
    type: String,
    required: true,
  },
  showCart: {
    type: Boolean,
    required: true,
  },
  isLoading: {
    type: Boolean,
    required: true,
  },
})

const emit = defineEmits([
  'update-search-query',
  'update-sort-by',
  'update-sort-order',
  'add-to-cart',
])

function onSearchInput(event) {
  emit('update-search-query', event.target.value)
}

function onSortByChange(event) {
  emit('update-sort-by', event.target.value)
}

function onSortOrderChange(event) {
  emit('update-sort-order', event.target.value)
}

function onAddToCart(lesson) {
  emit('add-to-cart', lesson)
}
</script>

<template>
  <div class="lessons-page" :class="{ active: !showCart }">
    <div class="controls-container">
      <div class="search-container">
        <input
          type="text"
          :value="searchQuery"
          @input="onSearchInput"
          placeholder="Search lessons by title or description..."
          class="search-bar"
        />
        <i class="fa-solid fa-magnifying-glass search-icon"></i>
      </div>

      <div class="sort-controls">
        <select
          :value="sortBy"
          @change="onSortByChange"
          class="sort-select"
        >
          <option value="">Sort by</option>
          <option value="title">Title</option>
          <option value="price">Price</option>
          <option value="availableInventory">Available</option>
        </select>

        <select
          :value="sortOrder"
          @change="onSortOrderChange"
          class="order-select"
          :disabled="!sortBy"
        >
          <option value="asc">Ascending</option>
          <option value="desc">Descending</option>
        </select>
      </div>
    </div>

      <div v-if="isLoading" class="loader-container">
        <div class="loader"></div>
        <p>Loading lessons...</p>
      </div>
      <main v-else>
      <figure v-for="lesson in lessons" :key="lesson._id">
        <img :src="lesson.image" :alt="'Cover of ' + lesson.title" />

        <div class="book-icon">
          <i class="fa-solid" :class="lesson.icon"></i>
        </div>

        <h2 v-text="lesson.title"></h2>
        <p v-html="lesson.description"></p>
        <p class="price">Price: ${{ lesson.price.toFixed(2) }}</p>
        <p>Available: {{ lesson.availableInventory }}</p>
        <button
          @click="onAddToCart(lesson)"
          :disabled="lesson.availableInventory === 0"
          class="add-to-cart-btn"
        >
          {{ lesson.availableInventory > 0 ? 'Add to cart' : 'Out of stock' }}
        </button>
      </figure>
      </main>
    </div>
  </div>
</template>

<style scoped>
.loader-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  color: #7f8c8d;
}

.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
