<template>
  <div class="container">
    <div class="product-card">
      <span class="badge">Trending</span>
      
      <div class="image-container">
        <span class="product-emoji">🎧</span>
      </div>
      
      <div class="product-details">
        <h2 class="product-title">{{ productName }}</h2>
        <p class="product-description">Premium sound quality with active noise cancellation.</p>
        
        <div class="price-row">
          <span class="price">${{ price }}</span>
          <span class="stock">In Stock</span>
        </div>

        <div class="counter-section">
          <span class="quantity-text">Items added: <strong>{{ quantity }}</strong></span>
        </div>
        
        <button class="cart-btn" @click="addToCart">
          <span>Add to Cart</span>
          <span class="btn-icon">→</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const productName = ref('Pro Wireless Headphones')
const price = ref(129)
const quantity = ref(0)

function addToCart() {
  quantity.value++
}
</script>

<style scoped>
/* တစ်ပြင်လုံးကို အလယ်ဗဟိုပို့ပြီး နောက်ခံအမည်းရောင် ပုံစံပြောင်းတာ */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 80vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* ကတ်ပြား ဒီဇိုင်းအသစ် (Modern Glassmorphism look) */
.product-card {
  background: #ffffff;
  border-radius: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  width: 320px;
  overflow: hidden;
  position: relative;
  transition: transform 0.3s ease;
}

.product-card:hover {
  transform: translateY(-5px);
}

/* Badge ဒီဇိုင်း */
.badge {
  position: absolute;
  top: 15px;
  left: 15px;
  background: #ff4757;
  color: white;
  padding: 5px 12px;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: bold;
  z-index: 10;
}

/* ပုံပြတဲ့နေရာ */
.image-container {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.product-emoji {
  font-size: 5rem;
  filter: drop-shadow(0 8px 8px rgba(0,0,0,0.15));
}

/* စာသားများနှင့် အောက်ခြေအပိုင်း */
.product-details {
  padding: 25px;
}

.product-title {
  margin: 0 0 8px 0;
  font-size: 1.4rem;
  color: #1e293b;
}

.product-description {
  font-size: 0.85rem;
  color: #64748b;
  margin: 0 0 20px 0;
  line-height: 1.4;
}

.price-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.price {
  font-size: 1.6rem;
  font-weight: 800;
  color: #0f172a;
}

.stock {
  color: #10b981;
  background: #ecfdf5;
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
}

.counter-section {
  background: #f8fafc;
  padding: 10px 15px;
  border-radius: 12px;
  margin-bottom: 20px;
  font-size: 0.9rem;
  color: #334155;
}

/* ခလုတ် အလန်းစား ဒီဇိုင်း */
.cart-btn {
  width: 100%;
  background: #0f172a;
  color: white;
  border: none;
  padding: 14px;
  border-radius: 14px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  transition: background 0.2s ease;
}

.cart-btn:hover {
  background: #334155;
}

.btn-icon {
  transition: transform 0.2s ease;
}

.cart-btn:hover .btn-icon {
  transform: translateX(5px);
}
</style>