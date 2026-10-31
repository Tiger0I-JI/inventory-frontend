<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'


const products = ref([])
const form = ref({ id: null, name: '', price: '', quantity: '' })
const isEditing = ref(false)


const API_URL = 'http://localhost:3000/products'

// 1. READ
const fetchProducts = async () => {
  try {
    const response = await axios.get(API_URL)
    products.value = response.data
  } catch (error) {
    console.error('Error fetching products:', error)
  }
}

// 2. CREATE & UPDATE
const submitForm = async () => {
  try {
    if (isEditing.value) {
      
      await axios.put(`${API_URL}/${form.value.id}`, {
        name: form.value.name,
        price: Number(form.value.price),
        quantity: Number(form.value.quantity)
      })
    } else {
      
      await axios.post(API_URL, {
        name: form.value.name,
        price: Number(form.value.price),
        quantity: Number(form.value.quantity)
      })
    }
    
    
    form.value = { id: null, name: '', price: '', quantity: '' }
    isEditing.value = false
    fetchProducts()
  } catch (error) {
    console.error('Error saving product:', error)
    alert('Failed to save product')
  }
}


const editProduct = (product) => {
  form.value = { ...product }
  isEditing.value = true
}


const cancelEdit = () => {
  form.value = { id: null, name: '', price: '', quantity: '' }
  isEditing.value = false
}

// 3. DELETE
const deleteProduct = async (id) => {
  if (confirm('Are you sure you want to delete this product?')) {
    try {
      await axios.delete(`${API_URL}/${id}`)
      fetchProducts()
    } catch (error) {
      console.error('Error deleting product:', error)
    }
  }
}


onMounted(() => {
  fetchProducts()
})
</script>

<template>
  <div class="container">
    <h1>📦 Product Inventory</h1>
    
    
    <div class="card">
      <form @submit.prevent="submitForm" class="product-form">
        <input v-model="form.name" type="text" placeholder="Product Name" required />
        <input v-model="form.price" type="number" placeholder="Price (THB)" required />
        <input v-model="form.quantity" type="number" placeholder="Quantity" required />
        
        <button type="submit" class="btn-primary">
          {{ isEditing ? 'Update Product' : 'Add Product' }}
        </button>
        <button type="button" v-if="isEditing" @click="cancelEdit" class="btn-secondary">
          Cancel
        </button>
      </form>
    </div>

   
    <table class="product-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Price</th>
          <th>Quantity</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in products" :key="product.id">
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>฿{{ Number(product.price).toLocaleString() }}</td>
          <td>{{ product.quantity }}</td>
          <td>
            <button @click="editProduct(product)" class="btn-edit">Edit</button>
            <button @click="deleteProduct(product.id)" class="btn-delete">Delete</button>
          </td>
        </tr>
        <tr v-if="products.length === 0">
          <td colspan="5" class="empty-state">No products found. Add some!</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.container { max-width: 900px; margin: 2rem auto; padding: 0 1rem; font-family: system-ui, sans-serif; }
h1 { color: #333; margin-bottom: 1.5rem; text-align: center; }
.card { background: #f8f9fa; padding: 1.5rem; border-radius: 8px; margin-bottom: 2rem; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.product-form { display: flex; gap: 10px; flex-wrap: wrap; }
.product-form input { flex: 1; min-width: 150px; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
button { padding: 10px 15px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; transition: opacity 0.2s; }
button:hover { opacity: 0.8; }
.btn-primary { background-color: #42b883; color: white; }
.btn-secondary { background-color: #6c757d; color: white; }
.btn-edit { background-color: #ffc107; color: #000; margin-right: 5px; }
.btn-delete { background-color: #dc3545; color: white; }
.product-table { width: 100%; border-collapse: collapse; background: white; }
.product-table th, .product-table td { padding: 12px; text-align: left; border-bottom: 1px solid #ddd; }
.product-table th { background-color: #343a40; color: white; }
.empty-state { text-align: center; color: #666; font-style: italic; }
</style>