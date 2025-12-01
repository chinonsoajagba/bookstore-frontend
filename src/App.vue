<script setup>
import { ref, computed, onMounted } from 'vue'
import HeaderBar from './components/HeaderBar.vue'
import LessonsPage from './components/LessonsPage.vue'
import CartPage from './components/CartPage.vue'

const sitename = ref('After School Lessons')
const searchQuery = ref('')
const sortBy = ref('')
const sortOrder = ref('asc')
const showCart = ref(false)
const orderConfirmed = ref(false)
const checkoutData = ref({
  name: '',
  phone: '',
})
const errors = ref({})
const products = ref([])
const cart = ref([])
const isLoading = ref(false)
const apiBaseUrl = 'https://bookstore-backend-isgt.onrender.com'

function getFallbackLessons() {
  return [
    {
      _id: '1001',
      title: 'Mathematics',
      description: 'A full course work of Advanced Mathematics.',
      price: 20.0,
      image: 'images/Maths1.jpg',
      availableInventory: 5,
      icon: 'fa-calculator',
      subject: 'Math',
    },
    {
      _id: '1002',
      title: 'English',
      description: 'Advanced Level English.',
      price: 10.0,
      image: 'images/english2.jpg',
      availableInventory: 5,
      icon: 'fa-book',
      subject: 'English',
    },
    {
      _id: '1003',
      title: 'Computer Science',
      description: 'Intro to Computer Science.',
      price: 8.0,
      image: 'images/computer.jpg',
      availableInventory: 5,
      icon: 'fa-laptop-code',
      subject: 'Computer Science',
    },
    {
      _id: '1004',
      title: 'International Relations',
      description: 'A full study guide on the intro to International Relations.',
      price: 6.0,
      image: 'images/inter-rel.jpg',
      availableInventory: 5,
      icon: 'fa-globe',
      subject: 'Politics',
    },
    {
      _id: '1005',
      title: 'Commerce',
      description: 'A full study guide to your future of Commerce.',
      price: 5.5,
      image: 'images/commerce.jpg',
      availableInventory: 12,
      icon: 'fa-chart-line',
      subject: 'Business',
    },
    {
      _id: '1006',
      title: 'Medicine',
      description: 'Beginner to Advanced level courses.',
      price: 12.0,
      image: 'images/medicine.jpg',
      availableInventory: 5,
      icon: 'fa-stethoscope',
      subject: 'Science',
    },
    {
      _id: '1007',
      title: 'Engineering',
      description: 'Advanced Engineering courses.',
      price: 15.0,
      image: 'images/engineering.jpg',
      availableInventory: 5,
      icon: 'fa-gears',
      subject: 'Engineering',
    },
    {
      _id: '1008',
      title: 'Law',
      description: 'Brand Law.',
      price: 9.0,
      image: 'images/law.jpg',
      availableInventory: 5,
      icon: 'fa-gavel',
      subject: 'Law',
    },
    {
      _id: '1009',
      title: 'Dark Magic',
      description: 'Shadow Wizard Money Gang.',
      price: 25.0,
      image: 'images/magic.jpg',
      availableInventory: 5,
      icon: 'fa-wand-magic-sparkles',
      subject: 'Mystical Arts',
    },
    {
      _id: '1010',
      title: 'Physics',
      description: 'Fundamental principles of Physics.',
      price: 18.0,
      image: 'images/physics.jpg',
      availableInventory: 8,
      icon: 'fa-atom',
      subject: 'Science',
    },
  ]
}

async function fetchLessons() {
  try {
    console.log('Fetching lessons from API...')
    const response = await fetch(`${apiBaseUrl}/lessons`)

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const lessons = await response.json()
    console.log('Lessons loaded from API:', lessons)
    
    // Map API fields to frontend expectations
    const mappedLessons = lessons.map(lesson => ({
      ...lesson,
      title: lesson.subject,
      availableInventory: lesson.spaces,
      image: `${apiBaseUrl}/images/${lesson.image}`,
      description: `A full course work of ${lesson.subject}. Location: ${lesson.location}`
    }))
    
    products.value = mappedLessons
  } catch (error) {
    console.error('Failed to fetch lessons from API:', error)
    console.log('Using fallback lessons data...')
    products.value = getFallbackLessons()
  } finally {
    isLoading.value = false
  }
}

function saveCartToStorage() {
  try {
    localStorage.setItem('bookstoreCart', JSON.stringify(cart.value))
  } catch (error) {
    console.error('Failed to save cart to storage:', error)
  }
}

function loadCartFromStorage() {
  try {
    const savedCart = localStorage.getItem('bookstoreCart')
    if (savedCart) {
      cart.value = JSON.parse(savedCart)
    }
  } catch (error) {
    console.error('Failed to load cart from storage:', error)
  }
}

function canAddToCart(book) {
  return book.availableInventory > 0
}

async function addToCart(lesson) {
  if (canAddToCart(lesson) && lesson.availableInventory > 0) {
    const cartItem = {
      id: lesson._id,
      price: lesson.price,
      title: lesson.title,
      uniqueId: `${lesson._id}_${Date.now()}`,
    }

    cart.value.push(cartItem)

    try {
      console.log('Updating inventory for lesson:', lesson._id)

      const response = await fetch(`${apiBaseUrl}/lessons/${lesson._id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          availableInventory: lesson.availableInventory - 1,
        }),
      })

      if (response.ok) {
        const updatedLesson = await response.json()
        lesson.availableInventory = updatedLesson.availableInventory
      } else {
        console.error('Failed to update inventory via API')
        if (lesson.availableInventory > 0) {
          lesson.availableInventory--
        }
      }

      saveCartToStorage()
    } catch (error) {
      console.error('Failed to update inventory:', error)
      if (lesson.availableInventory > 0) {
        lesson.availableInventory--
      }
      saveCartToStorage()
    }
  } else {
    console.log('Cannot add to cart - no inventory available')
  }
}

async function removeFromCart(cartItem) {
  console.log('Removing item:', cartItem)

  const itemIndex = cart.value.findIndex(
    (item) => item.id === cartItem.id && item === cartItem,
  )

  if (itemIndex !== -1) {
    cart.value.splice(itemIndex, 1)

    try {
      const lesson = products.value.find((p) => p._id === cartItem.id)
      if (lesson) {
        console.log('Restoring inventory for lesson:', lesson._id)

        const response = await fetch(`${apiBaseUrl}/lessons/${lesson._id}`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            availableInventory: lesson.availableInventory + 1,
          }),
        })

        console.log('Restore response status:', response.status)

        if (response.ok) {
          const updatedLesson = await response.json()
          lesson.availableInventory = updatedLesson.availableInventory
        } else {
          console.error('Failed to restore inventory via API')
          lesson.availableInventory++
        }
      }
      saveCartToStorage()
    } catch (error) {
      console.error('Failed to restore inventory:', error)
      const lesson = products.value.find((p) => p._id === cartItem.id)
      if (lesson) lesson.availableInventory++
      saveCartToStorage()
    }
  }
}

function toggleCart() {
  showCart.value = !showCart.value
  orderConfirmed.value = false
}

function getLessonTitle(lessonId) {
  const lesson = products.value.find((p) => p._id === lessonId)
  return lesson ? lesson.title : 'Unknown Lesson'
}

function validateCheckout() {
  errors.value = {}
  let isValid = true

  const nameRegex = /^[A-Za-z\s]+$/
  if (!checkoutData.value.name) {
    errors.value.name = 'Name is required'
    isValid = false
  } else if (!nameRegex.test(checkoutData.value.name)) {
    errors.value.name = 'Name must contain letters only'
    isValid = false
  }

  const phoneRegex = /^\d+$/
  if (!checkoutData.value.phone) {
    errors.value.phone = 'Phone is required'
    isValid = false
  } else if (!phoneRegex.test(checkoutData.value.phone)) {
    errors.value.phone = 'Phone must contain numbers only'
    isValid = false
  }

  return isValid
}

async function processCheckout() {
  if (validateCheckout()) {
    try {
      const orderData = {
        name: checkoutData.value.name,
        phone: checkoutData.value.phone,
        bookIDs: cart.value.map((item) => item.id),
        quantities: cart.value.map(() => 1),
        total: cartTotal.value,
      }

      console.log('Submitting order:', orderData)

      const response = await fetch(`${apiBaseUrl}/orders`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(orderData),
      })

      if (response.ok) {
        const orderResult = await response.json()
        console.log('Order created successfully:', orderResult)

        orderConfirmed.value = true

        setTimeout(() => {
          cart.value = []
          checkoutData.value = { name: '', phone: '' }
          showCart.value = false
          saveCartToStorage()
        }, 2000)
      } else {
        const errorText = await response.text()
        console.error('Order failed with status:', response.status, errorText)
        alert('Failed to submit order. Please try again.')
      }
    } catch (error) {
      console.error('Order submission failed:', error)
      alert('Order submission failed. Please check your connection.')
    }
  }
}

const filteredLessons = computed(() => {
  if (!searchQuery.value) {
    return products.value
  }
  const query = searchQuery.value.toLowerCase()
  return products.value.filter(
    (lesson) =>
      lesson.title.toLowerCase().includes(query) ||
      lesson.description.toLowerCase().includes(query),
  )
})

const sortedAndFilteredLessons = computed(() => {
  const lessons = [...filteredLessons.value]

  if (sortBy.value) {
    lessons.sort((a, b) => {
      let aVal = a[sortBy.value]
      let bVal = b[sortBy.value]

      if (typeof aVal === 'string') {
        aVal = aVal.toLowerCase()
        bVal = bVal.toLowerCase()
      }

      if (aVal < bVal) return sortOrder.value === 'asc' ? -1 : 1
      if (aVal > bVal) return sortOrder.value === 'asc' ? 1 : -1
      return 0
    })
  }

  return lessons
})

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + item.price, 0)
})

const isCheckoutValid = computed(() => {
  return (
    cart.value.length > 0 &&
    checkoutData.value.name &&
    checkoutData.value.phone &&
    /^[A-Za-z\s]+$/.test(checkoutData.value.name) &&
    /^\d+$/.test(checkoutData.value.phone)
  )
})

function handleUpdateSearchQuery(value) {
  searchQuery.value = value
}

function handleUpdateSortBy(value) {
  sortBy.value = value
}

function handleUpdateSortOrder(value) {
  sortOrder.value = value
}

function handleAddToCart(lesson) {
  addToCart(lesson)
}

function handleToggleCart() {
  toggleCart()
}

function handleRemoveFromCart(item) {
  removeFromCart(item)
}

function handleUpdateCheckoutData(data) {
  checkoutData.value = data
}

function handleSubmitCheckout() {
  processCheckout()
}

onMounted(() => {
  loadCartFromStorage()
  isLoading.value = true
  fetchLessons()
})
</script>

<template>
  <div>
    <HeaderBar
      :sitename="sitename"
      :cart-count="cart.length"
      :cart-total="cartTotal"
      :show-cart="showCart"
      :is-cart-disabled="cart.length === 0"
      @toggle-cart="handleToggleCart"
    />

    <LessonsPage
      :lessons="sortedAndFilteredLessons"
      :search-query="searchQuery"
      :sort-by="sortBy"
      :sort-order="sortOrder"
      :show-cart="showCart"
      :is-loading="isLoading"
      @update-search-query="handleUpdateSearchQuery"
      @update-sort-by="handleUpdateSortBy"
      @update-sort-order="handleUpdateSortOrder"
      @add-to-cart="handleAddToCart"
    />

    <CartPage
      :cart="cart"
      :cart-total="cartTotal"
      :checkout-data="checkoutData"
      :errors="errors"
      :order-confirmed="orderConfirmed"
      :show-cart="showCart"
      :is-checkout-valid="isCheckoutValid"
      :get-lesson-title="getLessonTitle"
      @toggle-cart="handleToggleCart"
      @remove-from-cart="handleRemoveFromCart"
      @update-checkout-data="handleUpdateCheckoutData"
      @submit-checkout="handleSubmitCheckout"
    />
  </div>
</template>
