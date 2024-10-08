<template>
  <div :class="themeClass">
    <!-- Header Section -->
    <header :class="['p-4 flex justify-between items-center', themeClass]">
      <div class="flex items-center">
        <h1 :class="headerTextClass">GoodMarting</h1>
      </div>
      <!-- Shopping Cart and Wishlist Icons -->
      <div class="flex space-x-4 p-4">
        <button @click="handleAuthButtonClick" :class="{ 'bg-red-500': loggedIn, 'bg-blue-500': !loggedIn }" class="text-white p-2 rounded-lg">
          {{ loggedIn ? 'Sign Out' : 'Login' }}
        </button>
        <button @click="redirectToCart" :class="['text-2xl', iconClass]">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
            <path d="M6 6h15l1 10H8L6 6zM4 4h2l3.6 9.6L10.2 16H20v2H8.4l-1.6-2H3V4h1z" />
          </svg>
        </button>
        <button @click="redirectToWishlist" :class="['text-2xl', iconClass]">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
            <path d="M12 3l1.5 3h3.5l-2.7 2.5L15 12l-3-2-3 2 1-3-2.7-2.5h3.5L12 3zm0 12c2.1 0 4-1.7 4-4s-1.9-4-4-4-4 1.7-4 4 1.9 4 4 4zm0 2c-4.4 0-8-3.6-8-8s3.6-8 8-8 8 3.6 8 8-3.6 8-8 8z" />
          </svg>
        </button>
        <button @click="goToComparison" :class="['text-2xl', buttonTextClass]">Compare Products</button>
        <!-- Theme Swap Button -->
        <button @click="swapTheme" :class="['text-2xl', buttonTextClass]">
          Swap Theme
        </button>
      </div>
    </header>

    <!-- Sorting and Filtering -->
    <SortControls :sortPrice="sortPrice" :sortType="sortType" :categories="categories" :isDefaultSort="isDefaultSort"
      @update:sortPrice="updateSortPrice" @update:sortType="updateSortType" @sort-change="handleSortChange" />

    <!-- Discounted Products -->
    <div v-if="isLoading" class="flex justify-center items-center h-full">
      <p class="text-[5vw]">Loading...</p>
    </div>
    <Discount/>

    <!-- Product List -->
    <div :class="['border-2', themeListBorderClass, 'p-4']">
      <div v-if="isLoading" class="flex justify-center items-center h-full">
        <p class="text-[5vw]">Loading...</p>
      </div>
      <ul v-else class="flex flex-wrap -mx-4">
        <li v-for="item in filteredItems" :key="item.id" class="flex flex-col w-full md:w-1/2 lg:w-1/4 p-4 min-h-[60vw] sm:min-h-[25vw] md:min-h-[20vw] lg:min-h-[15vw] cursor-pointer" :class="themeClass">
          <a :href="`/#/about?id=${item.id}&sortPrice=${sortPrice}&sortType=${sortType}&userId=${userId}`" class="block h-full">
            <div :class="['border p-4 rounded-lg shadow-sm h-full flex flex-col', themeClass]">
              <img :src="item.image" :alt="item.title" class="w-full max-h-[35vw] object-contain mb-4">
              <div class="flex flex-col flex-grow">
                <div class="mb-4">
                  <h2 :class="itemTitleClass">{{ item.title }}</h2>
                </div>
                <div class="flex flex-col mb-4">
                  <p :class="itemPriceClass">{{ '$' + item.price }}</p>
                  <p :class="itemCategoryClass">Category: {{ item.category }}</p>
                  <div class="flex items-center">
                    <span :class="itemRatingClass">Rating:</span>
                    <div class="flex items-center">
                      <!-- Star SVGs for Rating -->
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </a>
          <button @click="toggleComparison(item)" :class="['mt-auto py-2 px-4 rounded', themeButtonClass]">
            {{ isInComparison(item.id) ? 'Remove from Comparison' : 'Add to Comparison' }}
          </button>
          <button @click="addToCart(item.id)" :class="['mt-2 py-2 px-4 rounded', themeButtonClass]">Add to Cart</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import useProductList from './js/ProductList.js';
import SortControls from './SortControls.vue';
import Discount from './ProductDiscount.vue';
import { useRouter } from 'vue-router';

const {
  filteredItems,
  isLoading,
  sortPrice,
  sortType,
  categories,
  addToCart,
  fetchProducts,
  userId,
  handleSortChange,
  isDefaultSort,
  goToComparison,
  handleAuthButtonClick,
  redirectToCart,
  redirectToWishlist,
  toggleComparison,
  isInComparison,
  updateSortPrice,
  updateSortType,
  loggedIn
} = useProductList();

const currentTheme = ref(localStorage.getItem('theme') || 'light');

const swapTheme = () => {
  const newTheme = currentTheme.value === 'light' ? 'dark' : 'light';
  currentTheme.value = newTheme;
  localStorage.setItem('theme', newTheme);
  
  // Dispatch a custom event to notify other components about the theme change
  window.dispatchEvent(new Event('theme-changed'));
};

const themeClass = computed(() => {
  return currentTheme.value === 'light' ? 'bg-gray-100 text-gray-900' : 'bg-gray-900 text-gray-100';
});

const themeButtonClass = computed(() => {
  return currentTheme.value === 'light'
    ? 'bg-pink-400 border-2 border-pink-600 text-gray-900'
    : 'bg-amber-400 border-2 border-amber-600 text-gray-900';
});

const headerTextClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-pink-600 text-[3vw]' : 'text-amber-400 text-[3vw] ';
});

const buttonTextClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-pink-600' : 'text-amber-400';
});

const iconClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-pink-600' : 'text-amber-400';
});

const themeListBorderClass = computed(() => {
  return currentTheme.value === 'light' ? 'border-pink-500' : 'border-amber-500';
});

const itemTitleClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-pink-600' : 'text-amber-400';
});

const itemPriceClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-gray-900' : 'text-gray-100';
});

const itemCategoryClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-gray-600' : 'text-gray-300';
});

const itemRatingClass = computed(() => {
  return currentTheme.value === 'light' ? 'text-gray-600' : 'text-gray-300';
});

onMounted(() => {
  fetchProducts();
});
</script>

<style scoped>
/* Add any additional styles here */
</style>
