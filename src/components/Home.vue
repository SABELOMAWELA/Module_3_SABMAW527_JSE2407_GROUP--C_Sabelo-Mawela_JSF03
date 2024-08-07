<template>
  <div id="app">
    <navbar />
    <FilterSortComponent 
      :categories="categories" 
      @filter="handleFilter" 
      @search="handleSearch" 
      @sort="handleSort" 
    />
    <div v-if="loading">Loading...</div>
    <cards v-else :products="filteredProducts" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import navbar from '../components/navbar.vue';
import cards from '../components/cards.vue';
import FilterSortComponent from '../components/FilterSortComponent.vue';

const products = ref([]);
const filteredProducts = ref([]);
const categories = ref([]);
const searchTerm = ref('');
const selectedCategory = ref('');
const sorting = ref('');
const loading = ref(true);

/**
 * Fetches product data from the API and updates products, filteredProducts, and categories.
 * Sets loading to false after data is fetched or in case of an error.
 */
function fetchData() {
  axios.get('https://fakestoreapi.com/products')
    .then(res => {
      products.value = res.data;
      filteredProducts.value = res.data;
      categories.value = [...new Set(res.data.map(product => product.category))];
      loading.value = false; 
    })
    .catch(error => {
      console.error("There was an error fetching the data:", error);
      loading.value = false;
    });
}

onMounted(() => fetchData());

/**
 * Filters the products based on selectedCategory, searchTerm, and sorting.
 * Updates filteredProducts with the filtered results.
 */
function filterProducts() {
  let tempProducts = [...products.value];

  if (selectedCategory.value) {
    tempProducts = tempProducts.filter(product => product.category === selectedCategory.value);
  }

  if (searchTerm.value) {
    tempProducts = tempProducts.filter(product => product.title.toLowerCase().includes(searchTerm.value.toLowerCase()));
  }

  if (sorting.value) {
    if (sorting.value === 'Price: low to high') {
      tempProducts.sort((a, b) => a.price - b.price);
    } else if (sorting.value === 'Price: high to low') {
      tempProducts.sort((a, b) => b.price - a.price);
    }
  }

  filteredProducts.value = tempProducts;
}

/**
 * Handles the category filter change.
 * @param {string} category - The selected category to filter products by.
 */
function handleFilter(category) {
  selectedCategory.value = category;
  filterProducts();
}

/**
 * Handles the search term change.
 * @param {string} term - The search term to filter products by.
 */
function handleSearch(term) {
  searchTerm.value = term;
  filterProducts();
}

/**
 * Handles the sorting order change.
 * @param {string} order - The sorting order (e.g., 'Price: low to high', 'Price: high to low').
 */
function handleSort(order) {
  sorting.value = order;
  filterProducts();
}
</script>
