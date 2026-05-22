<template>
  <div class="app-container">
    <header class="main-header">
      <div class="header-left">
        <h1 class="logo"> TechStore</h1>
        <p class="tagline">The Ultimate Frontend Experience</p>
      </div>
      <div class="cart-status-wrapper">
        <div class="cart-status">
          <span class="cart-icon">🛒</span>
          <span class="cart-text">Cart Total</span>
          <strong class="badge">{{ totalCartItems }}</strong>
        </div>
      </div>
    </header>

    <div class="controls-bar">
      <div class="search-box-wrapper">
        <span class="search-icon">🔍</span>
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="Search items by name..." 
          class="search-input"
        />
      </div>
      <div class="filter-buttons">
        <button 
          v-for="cat in categories" 
          :key="cat"
          :class="['filter-btn', { active: selectedCategory === cat }]"
          @click="selectedCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
    </div>

    <main class="content-layout">
      
      <section class="products-section">
        <div class="section-title-row">
          <h2>Available Products</h2>
          <span class="results-count">{{ filteredProducts.length }} items found</span>
        </div>

        <div v-if="filteredProducts.length === 0" class="no-results">
          <p>😢 No products match your search or filter.</p>
        </div>

        <div v-else class="products-grid">
          <div v-for="product in filteredProducts" :key="product.id" class="product-card">
            <span class="product-tag">{{ product.category }}</span>
            
            <div class="product-image-box">
              <img :src="product.image" :alt="product.name" class="product-img" />
            </div>
            
            <div class="product-info">
              <h3>{{ product.name }}</h3>
              <p class="desc">{{ product.desc }}</p>
              <div class="price-row">
                <span class="price">${{ product.price }}</span>
                <button class="add-btn" @click="addToCart(product)">
                  Add to Cart
                </button>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section class="cart-section">
        <h2>Your Shopping Cart</h2>
        
        <div v-if="cart.length === 0" class="empty-cart">
          <div class="empty-icon">🛍️</div>
          <p>Your cart is empty</p>
          <span>Items you add will appear here.</span>
        </div>

        <div v-else class="cart-wrapper">
          <div class="cart-list">
            <div v-for="item in cart" :key="item.id" class="cart-item">
              <img :src="item.image" :alt="item.name" class="cart-item-img" />
              
              <div class="item-details">
                <h4>{{ item.name }}</h4>
                <div class="qty-controls">
                  <button class="qty-btn" @click="decreaseQty(item)">-</button>
                  <span class="qty-number">{{ item.qty }}</span>
                  <button class="qty-btn" @click="addToCart(item)">+</button>
                  <span class="item-price-total">/ ${{ item.price * item.qty }}</span>
                </div>
              </div>
              <button class="remove-btn" @click="removeFromCart(item)" title="Remove item">✕</button>
            </div>
          </div>

          <div class="cart-summary">
            <div class="summary-row">
              <span>Subtotal</span>
              <span>${{ totalAmount }}</span>
            </div>
            <div class="summary-row">
              <span>Shipping</span>
              <span class="free-text">FREE</span>
            </div>
            <hr class="divider" />
            <div class="total-row">
              <span>Total Amount:</span>
              <span class="total-price">${{ totalAmount }}</span>
            </div>
            <button class="checkout-btn" @click="checkout">
              Proceed to Checkout
            </button>
          </div>
        </div>
      </section>

    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// ၁။ Master Product Data List (ဓာတ်ပုံအစစ် Link များ အစားထိုးထားပါတယ်)
const products = ref([
  { 
    id: 1, 
    name: 'Pro Wireless Headphones', 
    price: 129, 
    image: 'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=500&q=80', 
    desc: 'Active noise cancellation with premium studio sound dynamic.', 
    category: 'Audio' 
  },
  { 
    id: 2, 
    name: 'Smart Fitness Watch', 
    price: 199, 
    image: 'https://images.unsplash.com/photo-1523275335684-37898b6baf30?w=500&q=80', 
    desc: 'AMOLED display, heart rate tracking and 14-day battery.', 
    category: 'Wearables' 
  },
  { 
    id: 3, 
    name: 'Ergonomic Wireless Mouse', 
    price: 49, 
    image: 'https://images.unsplash.com/photo-1615663245857-ac93bb7c39e7?w=500&q=80', 
    desc: 'Precision optical sensor with silent clicks and ergonomic grip.', 
    category: 'Accessories' 
  },
  { 
    id: 4, 
    name: 'Mechanical RGB Keyboard', 
    price: 89, 
    image: 'https://images.unsplash.com/photo-1587829741301-dc798b83add3?w=500&q=80', 
    desc: 'Hot-swappable tactile switches with customizable dynamic RGB.', 
    category: 'Accessories' 
  },
 { 
    id: 5, 
    name: 'Minimalist Desk Mat', 
    price: 29, 
    // ဒါက လုံးဝအသစ်လဲပေးထားတဲ့ စိတ်ချရတဲ့ လင့်ခ်ဖြစ်ပါတယ်
    image: 'https://images.unsplash.com/photo-1585776245991-cf89dd7fc73a?w=500&q=80', 
    desc: 'Waterproof smooth felt surface padding for workspace style.', 
    category: 'Accessories' 
  },
  { 
    id: 6, 
    name: 'Premium Soundbar', 
    price: 159, 
    image: 'https://images.unsplash.com/photo-1545454675-3531b543be5d?w=500&q=80', 
    desc: 'Crystal clear cinematic audio output with rich deep bass setup.', 
    category: 'Audio' 
  }
])

// ၂။ Filter / Search Inputs State
const searchQuery = ref('')
const selectedCategory = ref('All')
const categories = ['All', 'Audio', 'Wearables', 'Accessories']

// ၃။ Cart State
const cart = ref([])

// LOCAL STORAGE LOGIC
onMounted(() => {
  const savedCart = localStorage.getItem('techstore_cart')
  if (savedCart) {
    cart.value = JSON.parse(savedCart)
  }
})

watch(cart, (newCart) => {
  localStorage.setItem('techstore_cart', JSON.stringify(newCart))
}, { deep: true })


// ၄။ FUNCTIONS
function addToCart(product) {
  const existingItem = cart.value.find(item => item.id === product.id)
  if (existingItem) {
    existingItem.qty++
  } else {
    cart.value.push({ ...product, qty: 1 })
  }
}

function decreaseQty(item) {
  if (item.qty > 1) {
    item.qty--
  } else {
    removeFromCart(item)
  }
}

function removeFromCart(item) {
  cart.value = cart.value.filter(cartItem => cartItem.id !== item.id)
}

function checkout() {
  alert(`🎉 Thank you! Order placed successfully for $${totalAmount.value}`)
  cart.value = []
}

// ၅။ COMPUTED PROPERTIES
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const matchesSearch = product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchesCategory = selectedCategory.value === 'All' || product.category === selectedCategory.value
    return matchesSearch && matchesCategory
  })
})

const totalCartItems = computed(() => {
  return cart.value.reduce((total, item) => total + item.qty, 0)
})

const totalAmount = computed(() => {
  return cart.value.reduce((total, item) => total + (item.price * item.qty), 0)
})
</script>

<style scoped>
/* Core Styling */
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 30px 20px;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  background-color: #fafafa;
  min-height: 100vh;
}

/* Header UI */
.main-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 24px;
  border-bottom: 1px solid #e2e8f0;
  margin-bottom: 25px;
}
.logo { font-size: 2rem; font-weight: 800; color: #0f172a; margin: 0; letter-spacing: -0.5px; }
.tagline { margin: 4px 0 0 0; color: #64748b; font-size: 0.9rem; }
.cart-status {
  display: flex;
  align-items: center;
  gap: 10px;
  background: #ffffff;
  padding: 10px 18px;
  border-radius: 50px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.03);
  border: 1px solid #edf2f7;
}
.cart-text { font-weight: 600; font-size: 0.9rem; color: #334155; }
.badge { background: #2563eb; color: white; padding: 4px 12px; border-radius: 20px; font-size: 0.85rem; font-weight: 700; }

/* Filter & Search Bar Controls */
.controls-bar {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
  margin-bottom: 30px;
}
.search-box-wrapper {
  position: relative;
  flex: 1;
  min-width: 280px;
}
.search-icon { position: absolute; left: 14px; top: 12px; color: #94a3b8; }
.search-input {
  width: 100%;
  padding: 12px 12px 12px 42px;
  border-radius: 12px;
  border: 1px solid #cbd5e1;
  font-size: 0.95rem;
  box-sizing: border-box;
  outline: none;
  transition: all 0.2s;
}
.search-input:focus { border-color: #2563eb; box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.1); }

.filter-buttons { display: flex; gap: 8px; }
.filter-btn {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  padding: 10px 16px;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 500;
  font-size: 0.9rem;
  color: #475569;
  transition: all 0.2s;
}
.filter-btn:hover { background: #f1f5f9; }
.filter-btn.active { background: #0f172a; color: white; border-color: #0f172a; }

/* Responsive Grid Layout */
.content-layout {
  display: grid;
  grid-template-columns: 2.5fr 1.2fr;
  gap: 30px;
}
@media (max-width: 968px) {
  .content-layout { grid-template-columns: 1fr; }
}

.section-title-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
.section-title-row h2 { font-size: 1.4rem; color: #0f172a; margin: 0; }
.results-count { font-size: 0.85rem; color: #64748b; }

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 20px;
}

/* Beautiful Product Cards with Real Images */
.product-card {
  background: white;
  border-radius: 20px;
  border: 1px solid #e2e8f0;
  overflow: hidden;
  position: relative;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.02);
  transition: all 0.2s ease;
}
.product-card:hover { transform: translateY(-5px); box-shadow: 0 12px 20px rgba(0,0,0,0.06); }
.product-tag {
  position: absolute; top: 12px; left: 12px; background: rgba(15, 23, 42, 0.85);
  padding: 4px 12px; border-radius: 30px; font-size: 0.7rem; font-weight: 600; color: white;
  backdrop-filter: blur(4px); z-index: 2;
}
.product-image-box {
  height: 180px; width: 100%; overflow: hidden; background: #f1f5f9;
}
.product-img {
  width: 100%; height: 100%; object-fit: cover; transition: transform 0.3s ease;
}
.product-card:hover .product-img { transform: scale(1.05); }

.product-info { padding: 20px; }
.product-info h3 { margin: 0 0 6px 0; font-size: 1.15rem; color: #0f172a; }
.desc { font-size: 0.85rem; color: #64748b; margin: 0 0 20px 0; line-height: 1.4; min-height: 38px; }
.price-row { display: flex; justify-content: space-between; align-items: center; }
.price { font-size: 1.3rem; font-weight: 800; color: #0f172a; }
.add-btn {
  background: #2563eb; color: white; border: none; padding: 10px 16px;
  border-radius: 10px; cursor: pointer; font-weight: 600; font-size: 0.85rem; transition: background 0.2s;
}
.add-btn:hover { background: #1d4ed8; }
.no-results { text-align: center; padding: 60px 20px; color: #64748b; background: white; border-radius: 20px; border: 1px solid #e2e8f0; }

/* Advanced Cart Design */
.cart-section {
  background: #ffffff; padding: 24px; border-radius: 20px;
  border: 1px solid #e2e8f0; align-self: start; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.02);
}
.cart-section h2 { font-size: 1.3rem; margin: 0 0 20px 0; color: #0f172a; }
.empty-cart { text-align: center; color: #94a3b8; padding: 40px 10px; }
.empty-icon { font-size: 3rem; margin-bottom: 10px; }
.empty-cart p { margin: 0; font-weight: 600; color: #475569; }
.empty-cart span { font-size: 0.8rem; color: #94a3b8; }

.cart-list { max-height: 320px; overflow-y: auto; margin-bottom: 20px; padding-right: 5px; }
.cart-item {
  display: flex; align-items: center; gap: 12px; background: #f8fafc;
  padding: 12px; border-radius: 14px; margin-bottom: 12px; border: 1px solid #f1f5f9;
}
.cart-item-img {
  width: 50px; height: 50px; object-fit: cover; border-radius: 8px; background: #cbd5e1;
}
.item-details { flex-grow: 1; }
.item-details h4 { margin: 0 0 4px 0; font-size: 0.95rem; color: #1e293b; }
.qty-controls { display: flex; align-items: center; gap: 8px; }
.qty-btn {
  background: white; border: 1px solid #cbd5e1; border-radius: 6px;
  width: 24px; height: 24px; font-weight: bold; cursor: pointer; display: flex; align-items: center; justify-content: center;
}
.qty-btn:hover { background: #f1f5f9; }
.qty-number { font-size: 0.9rem; font-weight: 700; color: #0f172a; min-width: 15px; text-align: center; }
.item-price-total { font-size: 0.8rem; color: #64748b; font-weight: 500; }
.remove-btn { background: none; border: none; color: #94a3b8; cursor: pointer; font-size: 0.9rem; padding: 5px; }
.remove-btn:hover { color: #ef4444; }

/* Summary & Checkout Button */
.cart-summary { background: #f8fafc; padding: 16px; border-radius: 14px; border: 1px solid #f1f5f9; }
.summary-row { display: flex; justify-content: space-between; font-size: 0.9rem; color: #475569; margin-bottom: 8px; }
.free-text { color: #10b981; font-weight: 600; }
.divider { border: 0; border-top: 1px solid #e2e8f0; margin: 12px 0; }
.total-row { display: flex; justify-content: space-between; align-items: center; font-size: 1.05rem; font-weight: 700; color: #0f172a; margin-bottom: 15px; }
.total-price { font-size: 1.4rem; color: #2563eb; font-weight: 800; }
.checkout-btn {
  width: 100%; background: #0f172a; color: white; border: none; padding: 14px;
  border-radius: 12px; font-size: 0.95rem; font-weight: 600; cursor: pointer; transition: background 0.2s;
}
.checkout-btn:hover { background: #1e293b; }
</style>