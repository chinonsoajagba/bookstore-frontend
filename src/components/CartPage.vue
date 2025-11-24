<script setup>
const {
  cart,
  cartTotal,
  checkoutData,
  errors,
  orderConfirmed,
  showCart,
  isCheckoutValid,
  getBookTitle,
} = defineProps({
  cart: {
    type: Array,
    required: true,
  },
  cartTotal: {
    type: Number,
    required: true,
  },
  checkoutData: {
    type: Object,
    required: true,
  },
  errors: {
    type: Object,
    required: true,
  },
  orderConfirmed: {
    type: Boolean,
    required: true,
  },
  showCart: {
    type: Boolean,
    required: true,
  },
  isCheckoutValid: {
    type: Boolean,
    required: true,
  },
  getLessonTitle: {
    type: Function,
    required: true,
  },
})

const emit = defineEmits([
  'toggle-cart',
  'remove-from-cart',
  'update-checkout-data',
  'submit-checkout',
])

function onToggleCart() {
  emit('toggle-cart')
}

function onRemove(cartItem) {
  emit('remove-from-cart', cartItem)
}

function onNameInput(value) {
  emit('update-checkout-data', {
    ...checkoutData,
    name: value,
  })
}

function onPhoneInput(value) {
  emit('update-checkout-data', {
    ...checkoutData,
    phone: value,
  })
}

function onSubmitCheckout() {
  emit('submit-checkout')
}
</script>

<template>
  <div class="cart-page" :class="{ active: showCart }">
    <div class="controls-container">
      <button class="back-btn" @click="onToggleCart">
        <i class="fa-solid fa-arrow-left"></i>
        Back to Lessons
      </button>
      <h2>Shopping Cart</h2>
    </div>

    <main>
      <div v-if="cart.length === 0" class="empty-cart">
        <p>Your cart is empty</p>
      </div>

      <div v-else>
        <div v-for="item in cart" :key="item.id" class="cart-item">
          <div class="item-details">
            <h3>{{ getLessonTitle(item.id) }}</h3>
            <p class="price">${{ item.price.toFixed(2) }}</p>
          </div>
          <button @click="onRemove(item)" class="remove-btn">
            Remove
          </button>
        </div>

        <div class="cart-total">
          <h3>Total: ${{ cartTotal.toFixed(2) }}</h3>

          <div class="checkout-form">
            <h3>Checkout</h3>
            <div class="form-group">
              <input
                type="text"
                :value="checkoutData.name"
                @input="onNameInput($event.target.value)"
                placeholder="Full Name (letters only)"
                class="form-input"
              />
              <div class="error-message" v-if="errors.name">
                {{ errors.name }}
              </div>
            </div>

            <div class="form-group">
              <input
                type="text"
                :value="checkoutData.phone"
                @input="onPhoneInput($event.target.value)"
                placeholder="Phone Number (numbers only)"
                class="form-input"
              />
              <div class="error-message" v-if="errors.phone">
                {{ errors.phone }}
              </div>
            </div>

            <button
              @click="onSubmitCheckout"
              :disabled="!isCheckoutValid"
              class="checkout-submit-btn"
            >
              Submit Order
            </button>

            <div v-if="orderConfirmed" class="order-confirmation">
              <p>✅ Order submitted successfully!</p>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>
