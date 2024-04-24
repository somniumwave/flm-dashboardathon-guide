<template>
    <div id="token-prices" class="mt-4 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <div v-for="price in prices" :key="price.hash" 
            :class="['p-4 shadow rounded price-change', 
                        {'price-up': price.direction === 'up', 
                        'price-down': price.direction === 'down'}]">
            <p class="text-lg font-semibold">{{ price.unwrappedSymbol }}: {{ formatPrice(price.unwrappedSymbol, price.usd_price) }}</p>
        </div>
    </div>
</template>
  
  
<script>

import ApiClient from '../../../js/api-client'; // Adjust the path if necessary
import tokenData from '../../../data/tokens'; // Adjust path as necessary
  
  export default {
        name: 'TokenPrices',
        data() {
            return {
                prices: [], // This will store the token prices
                previousPrices: {} // Store previous prices for comparison
            };
        },
        methods: {
            async fetchTokenPrices() {
                const apiClient = new ApiClient();
                try {
                    const data = await apiClient.getFlamingoLivedataPricesLatest();
                    // Process each price and determine if it has gone up or down
                    this.prices = data.map(price => {
                        let direction = 'same'; // Default state
                        if (this.previousPrices[price.hash]) {
                            if (price.usd_price > this.previousPrices[price.hash]) {
                                direction = 'up';
                            } else if (price.usd_price < this.previousPrices[price.hash]) {
                                direction = 'down';
                            }
                        }
                        // Set timeout to clear the direction after 1 second
                        setTimeout(() => {
                            const index = this.prices.findIndex(p => p.hash === price.hash);
                            if (index !== -1) {
                                this.prices[index].direction = 'same'; // Direct assignment in Vue 3
                            }
                        }, 4000);
                        return { ...price, direction };
                    });

                    // Update previous prices
                    this.previousPrices = data.reduce((acc, price) => {
                        acc[price.hash] = price.usd_price;
                        return acc;
                    }, {});
                } catch (error) {
                    console.error('Error fetching token prices:', error);
                    this.prices = []; // Reset prices on error
                }
            },

            formatPrice(symbol, usd_price) {
                const token = tokenData[symbol];
                const tokensWithTwoDecimals = ['NEO', 'FLM', 'GAS', 'FUSD', 'GM', 'FLUND', 'USDL']; // Specify tokens that need two decimal places
                if (token) {
                    let decimalsToShow;
                    if (usd_price > 0 && usd_price < 0.00000001) {
                        decimalsToShow = 12; // Increase to 10 decimal places for very small numbers
                    } else if (tokensWithTwoDecimals.includes(symbol)) {
                        decimalsToShow = 3; // For specified tokens, show only two decimals
                    } else {
                        decimalsToShow = token.decimals; // Use the default decimals for other tokens
                    }
                    return new Intl.NumberFormat('en-US', {
                        style: 'currency',
                        currency: 'USD',
                        minimumFractionDigits: decimalsToShow,
                        maximumFractionDigits: decimalsToShow
                    }).format(usd_price);
                } else {
                    return `$${usd_price.toFixed(2)}`; // Fallback if token data is not found
                }
            }
        },
        created() {
            this.fetchTokenPrices(); // Fetch immediately when the component is created
            this.interval = setInterval(this.fetchTokenPrices, 15000); // Fetch every 15 seconds
        },
        beforeUnmount() {
            clearInterval(this.interval); // Clear the interval when the component is unmounted
        }
    }

</script>
  
<style scoped>
    .price-change {
        /* Transition settings apply only to the background-color property */
        transition-property: background-color;
        transition-duration: 4s; /* Slow fade-out */
        transition-timing-function: ease-out;
    }

    /* Immediate color application without transition */
    .price-up, .price-down {
        transition: none; /* No transition when entering this state */
    }

    .price-up {
        background-color: #ccffcc; /* Light green for price increase */
    }

    .price-down {
        background-color: #ffcccc; /* Light red for price decrease */
    }
</style>

  