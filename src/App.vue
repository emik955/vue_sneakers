<script setup>
import { ref, watch, provide, computed } from 'vue'
import axios from 'axios'

import Header from './components/Header.vue'
import DrawerView from './components/DrawerView.vue'


const cart = ref([]) 
const isCreatingOrder = ref(false)

const drawerOpen = ref(false)

const totalPrice = computed(
  () => cart.value.reduce((acc, item) => acc + item.price, 0)
) 

const sumTaxe = computed(
  () => ~~(totalPrice.value * 0.05)
)

const closeDrawer = () => {
  drawerOpen.value = false
}

const openDrawer = () => {
  drawerOpen.value = true
}

const addToCart = (item) => {
  cart.value.push(item)
  item.isAdded = true
}

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1)
  item.isAdded = false
}

const createOrder = async () => {
  try {
    isCreatingOrder.value = true
    const { data } = await axios.post(`https://604781a0efa572c1.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: totalPrice.value
    })
    cart.value = []
    
    return data
  } catch (err) {
    cart.value = []
    console.log(err)
  } finally {
    isCreatingOrder.value = false
  }
}

watch(cart, () => {
  localStorage.setItem('cart', JSON.stringify(cart.value))
}, { deep: true })

provide('cart', {
  cart,
  closeDrawer,
  openDrawer,
  addToCart,
  removeFromCart
})

</script>

<template>
  <DrawerView 
    v-if="drawerOpen" 
    @closeDrawer="closeDrawer" 
    :totalPrice="totalPrice" 
    :sumTaxe="sumTaxe"
    @createOrder="createOrder"
    :isCreatingOrder="isCreatingOrder"
  />
  <div class="w-4/5 m-auto bg-white rounded-xl shadow-xl mt-14">
    <Header @openDrawer="openDrawer" :totalPrice="totalPrice"/>

    <div class="p-10">
      <router-view></router-view>
      <!-- <HomeView /> -->
    </div>
  </div>
</template>