<script setup>
const { sitename, cartCount, cartTotal, showCart, isCartDisabled } = defineProps({
  sitename: {
    type: String,
    required: true,
  },
  cartCount: {
    type: Number,
    required: true,
  },
  cartTotal: {
    type: Number,
    required: true,
  },
  showCart: {
    type: Boolean,
    required: true,
  },
  isCartDisabled: {
    type: Boolean,
    required: true,
  },
})

const emit = defineEmits(['toggle-cart'])

function onToggleCart() {
  emit('toggle-cart')
}
</script>

<template>
  <header>
    {{ sitename }}
    <button
      class="checkout-btn"
      @click="onToggleCart"
      :disabled="isCartDisabled"
    >
      <div class="cart-info">
        <span>{{ cartCount }}</span>
        <span>${{ cartTotal.toFixed(2) }}</span>
      </div>
      <span class="fa-solid fa-cart-shopping"></span>
      {{ showCart ? 'Back to Books' : 'View Cart' }}
    </button>
  </header>
</template>
