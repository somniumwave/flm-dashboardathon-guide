<template>
    <div class="flex flex-col items-center justify-center min-h-screen bg-gray-900">
      <div class="p-4 shadow-lg rounded bg-white w-full max-w-4xl">
        <h1 class="text-xl font-bold text-center">My AWESOME DeFi Dashboard</h1>
        <TokenPrices/>
      </div>
    </div>
  </template>
  
  <script>
  import TokenPrices from './TokenPrices.vue';
  
  export default {
    name: 'FlamingoDashboard',
    components: {
      TokenPrices
    }
  }
  </script>