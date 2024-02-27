<script setup>
import { inject, reactive, watch, ref, onMounted } from 'vue'
import axios from 'axios'
import debounce from 'lodash.debounce'
import CardList from '../components/CardList.vue'

const {cart, addToCart, removeFromCart} = inject('cart')

const items = ref([])

const filters = reactive({
  sortBy: 'title',
  searcQuery: ''
})

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const onChangeSelect = (e) => {
  filters.sortBy = e.target.value 
}

const onChangeSearchInput = debounce((e) => {
  filters.searcQuery = e.target.value
}, 200)

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        // parentId: item.id,
        item_id: item.id,
        item
      }

      item.isFavorite = true
      // const { data } = await axios.get(`https://604781a0efa572c1.mokky.dev/favorites`, obj)
      const { data } = await axios.post(`https://604781a0efa572c1.mokky.dev/favorites`, obj)

      item.favoriteId = data.id
    } else {
      item.isFavorite = false
      await axios.delete(`https://604781a0efa572c1.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios // Чтобы получить именно массив, - делаем деструктуризацию { data }
    .get(`https://604781a0efa572c1.mokky.dev/favorites`)

    items.value = items.value.map(item => {
      // const favorite = favorites.find(favorite => favorite.parentId === item.id)
      const favorite = favorites.find(favorite => favorite.item_id === item.id)
      
      if (!favorite) {
        return item
      }
      
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (err) {
    console.log(err)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searcQuery) {
      params.title = `*${filters.searcQuery}*`
    }

    const { data } = await axios // Чтобы получить именно массив, - делаем деструктуризацию { data }
    .get(`https://604781a0efa572c1.mokky.dev/items`,
      {
        params
      }
    )

    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    })).slice(0, 12) 
  } catch (err) {
    console.log(err)
  }
} 

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : [] 
  
  await fetchItems()
  await fetchFavorites() 

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

watch(filters, fetchItems)
</script>

<template>
  <div class="flex justify-between items-center">
        <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

        <div class="flex gap-4">
          <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
            <option value="name">По названию</option>
            <option value="price">По цене (дешевые)</option>
            <option value="-price">По цене (дорогие)</option>
          </select>
          <div class="relative">
            <img class="absolute left-3 top-3.5" src="/search.svg" alt="logo">
            <input
              @input="onChangeSearchInput"
              type="text"
              placeholder="Поиск..."
              class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400">
          </div>
        </div>
      </div>

      <div class="mt-10">
        <CardList :items="items" @addToFavorite="addToFavorite" @addToCart="onClickAddPlus"/>
      </div>
</template>