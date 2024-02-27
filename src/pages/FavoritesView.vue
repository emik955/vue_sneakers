<script setup>
  import { ref, onMounted  } from 'vue'
  import axios from 'axios'

  import CardList from '../components/CardList.vue'

  const favorites = ref([])

  onMounted(async () => {
    try {
      const { data } = await axios.get(
        'https://604781a0efa572c1.mokky.dev/favorites?_relations=items'
        // 'https://604781a0efa572c1.mokky.dev/favorites'
      )

      favorites.value = data.id
    } catch (err) {
      console.log(err)
    }
  })

</script>

<template>
  <CardList :items="favorites" isFavorites/>
</template>