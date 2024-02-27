<script setup>
  import { inject } from 'vue';
  import CartItem from '@/components/CartItem.vue'

  const { cart, removeFromCart } = inject('cart')
  console.log(cart)

</script>

<template>
  <div 
    class="flex flex-col flex-1 gap-4 justify-between"
    v-auto-animate
  >
    <CartItem 
      v-for="item in cart" 
      :key="item.id"
      :title="item.title"
      :price="item.price"
      :imageURL="item.imageUrl"
      @onClickRemove="() => removeFromCart(item)"
    />
  </div>
</template>
