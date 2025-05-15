<template>
  <div class="cashier-app">
    <div class="bg-pattern"></div>
    <div class="bg-gradient"></div>
    
    <header class="header">
      <div class="header-content">
        <img alt="Store logo" class="logo" src="./assets/blossom.png" width="60" height="60" />
        <h1 class="store-name">Sweet Cashier</h1>
      </div>
    </header>

    <main class="main-content">
      <div class="container">
        <div class="left-panel">
          <div class="categories glass-panel">
            <h2>Categories</h2>
            <div class="category-buttons">
              <button 
                v-for="category in categories" 
                :key="category.id"
                @click="selectCategory(category)"
                :class="['category-btn', selectedCategory === category ? 'active' : '']"
              >
                {{ category.name }}
              </button>
            </div>
          </div>
          
          <div class="products glass-panel">
            <h2>Products</h2>
            <div class="product-grid">
              <div 
                v-for="product in filteredProducts" 
                :key="product.id"
                class="product-card"
                @click="addToCart(product)"
              >
                <div class="product-image">
                  <img :src="product.image" :alt="product.name">
                </div>
                <div class="product-info">
                  <h3>{{ product.name }}</h3>
                  <p class="price">{{ formatCurrency(product.price) }}</p>
                </div>
                <div class="add-icon">+</div>
              </div>
            </div>
          </div>
        </div>
        
        <div class="right-panel">
          <div class="cart glass-panel">
            <h2>Shopping Cart</h2>
            
            <div class="cart-items" v-if="cart.length > 0">
              <div v-for="(item, index) in cart" :key="index" class="cart-item">
                <div class="item-image">
                  <img :src="item.image" :alt="item.name">
                </div>
                <div class="item-details">
                  <h3>{{ item.name }}</h3>
                  <p>{{ formatCurrency(item.price) }}</p>
                </div>
                <div class="item-quantity">
                  <button @click="decreaseQuantity(index)" class="qty-btn">-</button>
                  <span>{{ item.quantity }}</span>
                  <button @click="increaseQuantity(index)" class="qty-btn">+</button>
                </div>
                <div class="item-total">
                  {{ formatCurrency(item.price * item.quantity) }}
                </div>
                <button @click="removeItem(index)" class="remove-btn">×</button>
              </div>
            </div>
            
            <div class="empty-cart" v-else>
              <p>Your cart is empty</p>
              <div class="empty-cart-icon">🛒</div>
            </div>
            
            <div class="cart-summary">
              <div class="summary-line">
                <span>Subtotal:</span>
                <span>{{ formatCurrency(subtotal) }}</span>
              </div>
              <div class="summary-line">
                <span>Tax (10%):</span>
                <span>{{ formatCurrency(tax) }}</span>
              </div>
              <div class="summary-line total">
                <span>Total:</span>
                <span>{{ formatCurrency(total) }}</span>
              </div>
            </div>
            
            <div class="checkout-actions">
              <button @click="clearCart" class="clear-btn">Clear Cart</button>
              <button @click="checkout" class="checkout-btn">Checkout</button>
            </div>
          </div>
        </div>
      </div>
    </main>
    
    <div v-if="showCheckoutModal" class="modal">
      <div class="modal-content glass-panel">
        <h2>Payment Complete</h2>
        <p>Thank you for your purchase!</p>
        <div class="receipt">
          <h3>Receipt</h3>
          <div v-for="(item, index) in cart" :key="index" class="receipt-item">
            <span>{{ item.name }} × {{ item.quantity }}</span>
            <span>{{ formatCurrency(item.price * item.quantity) }}</span>
          </div>
          <div class="receipt-total">
            <span>Total:</span>
            <span>{{ formatCurrency(total) }}</span>
          </div>
        </div>
        <button @click="closeModal" class="close-modal-btn">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import strawberry from '@/assets/strawberry.jpeg'
import bubbletea from '@/assets/bubbletea.jpeg'
import saladbowl from '@/assets/saladbowl.jpeg'
import hairclip from '@/assets/hairclip.jpeg'
import sandwich from '@/assets/sandwich.jpeg'
import lavenderlatte from '@/assets/lavenderlatte.jpeg'
import cupcake from '@/assets/cupcake.jpeg'
import boucket from '@/assets/boucket.jpeg'
// Sample data
const categories = ref([
  { id: 1, name: 'All' },
  { id: 2, name: 'Food' },
  { id: 3, name: 'Drinks' },
  { id: 4, name: 'Desserts' },
  { id: 5, name: 'Accessories' }
]);

const selectedCategory = ref(categories.value[0]);

const products = ref([
  { 
    id: 1, 
    name: 'Strawberry Cake', 
    price: 35000, 
    category: 'Desserts',
    image: strawberry
  },
  { 
    id: 2, 
    name: 'Bubble Tea', 
    price: 25000, 
    category: 'Drinks',
    image: bubbletea  
  },
  { 
    id: 3, 
    name: 'Salad Bowl', 
    price: 30000, 
    category: 'Food',
    image: saladbowl
  },
  { 
    id: 4, 
    name: 'Hair Clip', 
    price: 15000, 
    category: 'Accessories',
    image: hairclip
  },
  { 
    id: 5, 
    name: 'Sandwich', 
    price: 28000, 
    category: 'Food',
    image: sandwich
  },
  { 
    id: 6, 
    name: 'Lavender Latte', 
    price: 32000, 
    category: 'Drinks',
    image: lavenderlatte
  },
  { 
    id: 7, 
    name: 'Cupcake', 
    price: 15000, 
    category: 'Desserts',
    image: cupcake
  },
  { 
    id: 8, 
    name: 'Boucket', 
    price: 45000, 
    category: 'Accessories',
    image: boucket
  },
]);

// Shopping cart
const cart = ref([]);
const showCheckoutModal = ref(false);

// Computed properties
const filteredProducts = computed(() => {
  if (selectedCategory.value.name === 'All') {
    return products.value;
  } else {
    return products.value.filter(product => product.category === selectedCategory.value.name);
  }
});

const subtotal = computed(() => {
  return cart.value.reduce((sum, item) => sum + (item.price * item.quantity), 0);
});

const tax = computed(() => {
  return subtotal.value * 0.1;
});

const total = computed(() => {
  return subtotal.value + tax.value;
});

// Methods
function selectCategory(category) {
  selectedCategory.value = category;
}

function addToCart(product) {
  const existingItem = cart.value.find(item => item.id === product.id);
  
  if (existingItem) {
    existingItem.quantity += 1;
  } else {
    cart.value.push({
      ...product,
      quantity: 1
    });
  }
}

function removeItem(index) {
  cart.value.splice(index, 1);
}

function increaseQuantity(index) {
  cart.value[index].quantity += 1;
}

function decreaseQuantity(index) {
  if (cart.value[index].quantity > 1) {
    cart.value[index].quantity -= 1;
  } else {
    removeItem(index);
  }
}

function clearCart() {
  cart.value = [];
}

function checkout() {
  if (cart.value.length > 0) {
    showCheckoutModal.value = true;
  }
}

function closeModal() {
  showCheckoutModal.value = false;
  clearCart();
}

function formatCurrency(value) {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    minimumFractionDigits: 0
  }).format(value);
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@400;500;600;700&display=swap');

:root {
  --primary-color: #ff85a2;
  --primary-light: #ffd6e0;
  --secondary-color: #a17fe0;
  --secondary-light: #c3a8e9;
  --accent1: #60dcb5;
  --accent2: #ffcf77;
  --text-color: #5a3d5c;
  --text-light: #7d6b7e;
  --border-radius: 18px;
  --box-shadow: 0 8px 32px rgba(31, 38, 135, 0.15);
}

.cashier-app {
  font-family: 'Quicksand', sans-serif;
  color: var(--text-color);
  min-height: 100vh;
  padding-bottom: 2rem;
  position: relative;
  overflow-x: hidden;
}

/* Beautiful Backgrounds */
.bg-pattern {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: 
    radial-gradient(#ff85a2 1px, transparent 1px),
    radial-gradient(#60dcb5 1px, transparent 1px);
  background-size: 40px 40px;
  background-position: 0 0, 20px 20px;
  opacity: 0.07;
  z-index: -2;
}

.bg-gradient {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #ffe6eb 0%, #f4edff 50%, #e5fbf5 100%);
  z-index: -1;
}

/* Glassmorphism Styles */
.glass-panel {
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(10px);
  border-radius: var(--border-radius);
  border: 1px solid rgba(255, 255, 255, 0.6);
  box-shadow: var(--box-shadow);
  transition: transform 0.3s, box-shadow 0.3s;
}

.glass-panel:hover {
  box-shadow: 0 8px 40px rgba(31, 38, 135, 0.2);
}

.header {
  margin-bottom: 2rem;
  padding: 1rem 2rem;
  position: sticky;
  top: 0;
  z-index: 10;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 20px rgba(255, 133, 162, 0.15);
}

.header-content {
  display: flex;
  align-items: center;
  max-width: 1400px;
  margin: 0 auto;
}

.logo {
  margin-right: 1rem;
  filter: drop-shadow(0 2px 4px rgba(255, 133, 162, 0.3));
}

.store-name {
  font-size: 1.8rem;
  color: var(--primary-color);
  margin: 0;
  font-weight: 700;
  text-shadow: 0 2px 4px rgba(255, 133, 162, 0.2);
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.container {
  display: flex;
  max-width: 1400px;
  margin: 0 auto;
  gap: 2rem;
  padding: 0 1rem;
}

.left-panel {
  flex: 1.5;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.right-panel {
  flex: 1;
}

h2 {
  color: var(--primary-color);
  border-bottom: 2px dashed var(--primary-light);
  padding-bottom: 0.75rem;
  margin-top: 0.5rem;
  margin-bottom: 1.25rem;
  font-weight: 700;
  font-size: 1.5rem;
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.categories, .products, .cart {
  padding: 1.5rem;
}

.category-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.category-btn {
  background: linear-gradient(to right, rgba(255, 214, 224, 0.7), rgba(195, 168, 233, 0.7));
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 50px;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s;
  color: var(--text-color);
  font-weight: 600;
  box-shadow: 0 3px 10px rgba(255, 133, 162, 0.2);
}

.category-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255, 133, 162, 0.3);
}

.category-btn.active {
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.7);
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 1.5rem;
}

.product-card {
  background: rgba(255, 255, 255, 0.8);
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(31, 38, 135, 0.1);
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
  position: relative;
  border: 1px solid rgba(255, 255, 255, 0.7);
}

.product-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 24px rgba(31, 38, 135, 0.15);
}

.product-card:after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 20px;
  background: linear-gradient(to bottom right, 
    rgba(255, 133, 162, 0.2), 
    rgba(161, 127, 224, 0.1));
  opacity: 0;
  transition: opacity 0.3s;
  pointer-events: none;
}

.product-card:hover:after {
  opacity: 1;
}

.product-image {
  height: 150px;
  overflow: hidden;
}

.product-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s;
}

.product-card:hover .product-image img {
  transform: scale(1.1);
}

.product-info {
  padding: 1.25rem;
  text-align: center;
  position: relative;
}

.product-info h3 {
  margin: 0 0 0.5rem;
  font-size: 1.1rem;
  color: var(--text-color);
  font-weight: 700;
}

.price {
  font-weight: bold;
  color: var(--primary-color);
  margin: 0;
  font-size: 1.1rem;
}

.add-icon {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 28px;
  height: 28px;
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: bold;
  box-shadow: 0 3px 8px rgba(255, 133, 162, 0.4);
  opacity: 0;
  transform: scale(0.8);
  transition: opacity 0.3s, transform 0.3s;
}

.product-card:hover .add-icon {
  opacity: 1;
  transform: scale(1);
}

.cart {
  position: sticky;
  top: 5.5rem;
}

.empty-cart {
  text-align: center;
  padding: 2rem 0;
  color: var(--text-light);
}

.empty-cart-icon {
  font-size: 3rem;
  margin-top: 1rem;
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.cart-items {
  max-height: 400px;
  overflow-y: auto;
  margin-bottom: 1.5rem;
  padding-right: 0.5rem;
}

.cart-items::-webkit-scrollbar {
  width: 5px;
}

.cart-items::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.5);
  border-radius: 10px;
}

.cart-items::-webkit-scrollbar-thumb {
  background: linear-gradient(to bottom, #ff85a2, #a17fe0);
  border-radius: 10px;
}

.cart-item {
  display: flex;
  align-items: center;
  padding: 0.75rem 0;
  border-bottom: 1px dashed var(--primary-light);
  position: relative;
}

.item-image {
  width: 50px;
  height: 50px;
  border-radius: 12px;
  overflow: hidden;
  margin-right: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.7);
  box-shadow: 0 3px 8px rgba(31, 38, 135, 0.1);
}

.item-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.item-details {
  flex: 1;
}

.item-details h3 {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 600;
}

.item-details p {
  margin: 0.25rem 0 0;
  font-size: 0.8rem;
  color: var(--primary-color);
}

.item-quantity {
  display: flex;
  align-items: center;
  margin: 0 1rem;
}

.qty-btn {
  width: 25px;
  height: 25px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(to right, #ff85a2, #ff9eb6);
  color: white;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 6px rgba(255, 133, 162, 0.3);
  transition: all 0.2s;
}

.qty-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 3px 8px rgba(255, 133, 162, 0.4);
}

.item-quantity span {
  margin: 0 0.6rem;
  width: 20px;
  text-align: center;
  font-weight: 600;
}

.item-total {
  font-weight: bold;
  margin-right: 1.5rem;
  color: var(--secondary-color);
}

.remove-btn {
  background: none;
  border: none;
  color: #ff4757;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0;
  line-height: 1;
  transition: transform 0.2s;
}

.remove-btn:hover {
  transform: scale(1.2);
}

.cart-summary {
  background: rgba(255, 255, 255, 0.5);
  border-radius: var(--border-radius);
  padding: 1.25rem;
  margin-bottom: 1.5rem;
  border: 1px dashed var(--primary-light);
}

.summary-line {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.75rem;
  font-weight: 500;
}

.summary-line.total {
  margin-top: 0.75rem;
  padding-top: 0.75rem;
  border-top: 2px dashed var(--primary-light);
  font-weight: 700;
  font-size: 1.15rem;
  color: var(--secondary-color);
}

.checkout-actions {
  display: flex;
  gap: 1rem;
}

.clear-btn, .checkout-btn {
  padding: 0.85rem;
  border: none;
  border-radius: 50px;
  font-weight: 600;
  cursor: pointer;
  flex: 1;
  transition: all 0.3s;
  font-size: 1.05rem;
  box-shadow: 0 4px 12px rgba(31, 38, 135, 0.15);
}

.clear-btn {
  background: rgba(255, 255, 255, 0.7);
  color: var(--text-color);
  border: 1px solid rgba(255, 255, 255, 0.8);
}

.clear-btn:hover {
  background: rgba(255, 255, 255, 0.9);
  transform: translateY(-3px);
  box-shadow: 0 6px 15px rgba(31, 38, 135, 0.2);
}

.checkout-btn {
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.checkout-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(255, 133, 162, 0.4);
}

/* Modal */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal-content {
  padding: 2.5rem;
  max-width: 500px;
  width: 100%;
  text-align: center;
  position: relative;
  overflow: hidden;
}

.modal-content:before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg, 
    rgba(255, 133, 162, 0.1) 0%, 
    rgba(255, 255, 255, 0.2) 50%, 
    rgba(161, 127, 224, 0.1) 100%);
  animation: shimmer 3s linear infinite;
  z-index: -1;
  pointer-events: none;
}

@keyframes shimmer {
  0% {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  100% {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

.receipt {
  background: rgba(255, 255, 255, 0.8);
  padding: 1.5rem;
  border-radius: var(--border-radius);
  margin: 1.5rem 0;
  text-align: left;
  box-shadow: 0 4px 15px rgba(31, 38, 135, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.7);
}

.receipt h3 {
  margin-top: 0;
  color: var(--primary-color);
  text-align: center;
  font-weight: 700;
  border-bottom: 1px dashed var(--primary-light);
  padding-bottom: 0.5rem;
  margin-bottom: 1rem;
}

.receipt-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.75rem;
  font-size: 0.95rem;
}

.receipt-total {
  display: flex;
  justify-content: space-between;
  margin-top: 1rem;
  padding-top: 0.75rem;
  border-top: 1px dashed var(--primary-light);
  font-weight: bold;
  color: var(--secondary-color);
}

.close-modal-btn {
  background: linear-gradient(to right, #ff85a2, #a17fe0);
  color: white;
  border: none;
  border-radius: 50px;
  padding: 0.85rem 2.5rem;
  font-weight: 600;
  cursor: pointer;
  margin-top: 1.5rem;
  font-size: 1.05rem;
  box-shadow: 0 4px 15px rgba(255, 133, 162, 0.3);
  transition: all 0.3s;
}

.close-modal-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(255, 133, 162, 0.4);
}

/* Responsive */
@media (max-width: 1024px) {
  .container {
    flex-direction: column;
  }
  
  .right-panel {
    flex: auto;
  }
  
  .cart {
    position: static;
  }
}

@media (max-width: 600px) {
  .product-grid {
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  }
  
  .category-buttons {
    justify-content: center;
  }
  
  .header-content {
    justify-content: center;
  }
}
</style>