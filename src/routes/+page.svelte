<script lang="ts">
	import CartItem from './cart-item.svelte';
	import ShoppingCart from 'phosphor-svelte/lib/ShoppingCart';
	import X from 'phosphor-svelte/lib/X';
	import type { CartProduct } from '$lib/types';

	let { data } = $props();
	let cartOpen = $state(false);
	let cartProducts = $state<CartProduct[]>([]);
	// let cartQuantityTotal = $derived(cartProducts.reduce((acc, cartProduct) => acc + cartProduct.quantity, 0));
	// const cartQuantity = $derived.by(()=> {
	// 	let total = 0;
	// 	for (const product of cartProducts) {
	// 		total += product.quantity;
	// 	}
	// 	return total;
	// });
	const cartStats = $derived.by(()=> {
		let totalQuantity = 0;
		let totalPrice = 0;
		for (const product of cartProducts) {
			totalQuantity += product.quantity;
			totalPrice += product.quantity * product.product.price;
		}
		return {
			totalQuantity,
			totalPrice
		};
	});
	const removeFromCart = (id: string) => {
		cartProducts = cartProducts.filter((product) => product.id !== id);
	};

	// use a derived state to check for a change to the status of free shipping
	// rather than an effect because it will run every time the price changes above 50

	const qualifiesForFreeShipping = $derived(cartStats.totalPrice >= 50);

	$effect(()=> {
		// if(cartStats.totalPrice > 50) {
		// 	alert('You have qualified for free shipping yo.');
		// }
		if(qualifiesForFreeShipping) {
			alert('You have qualified for free shipping yo.');
		}
	})

	$effect(()=> {
		if(cartStats.totalQuantity === 0 && cartOpen) {
			alert('Your cart is empty. Add items to your cart!!!!!');
		}
	})
</script>

<div class="flex items-center bg-gray-300 p-4">
	<span class="text-lg font-bold">SvelteMart</span>
	<div class="relative ml-auto flex items-center">
		<button onclick={() => (cartOpen = !cartOpen)} class="flex items-center rounded-full bg-sky-600 px-4 py-2 text-white hover:bg-sky-700">
			<ShoppingCart class="mr-2 size-5" />
			<span>Cart ({cartStats.totalQuantity})</span>
		</button>
		{#if cartOpen}
		<div class="absolute right-0 top-8 z-10 mt-2 w-80 rounded-lg bg-white shadow-xl">
			<div class="relative p-4">
				<h2 class="mb-4 text-lg font-semibold">Your Cart</h2>
				<button onclick={() => (cartOpen = false)} class="absolute right-4 top-4 rounded-full p-1 hover:bg-gray-100">
					<X class="size-4" />
				</button>
				{#if cartProducts.length > 0}	
				{#each cartProducts as _, index}
					<CartItem bind:cartProduct={cartProducts[index]} {removeFromCart} />
				{/each}
				<div class="mt-4 border-gray-200 pt-4">
					<p class="text-lg font-semibold">Total: ${cartStats.totalPrice.toFixed(2)}</p>
				</div>
				{/if}
			</div>
		</div>
		{/if}
	</div>
</div>

<div class="grid grid-cols-1 gap-6 bg-gray-100 p-8 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
	{#each data.products as product}
		<div class="overflow-hidden rounded-xl bg-white shadow-lg">
			<img src={product.thumbnail} alt={product.title} class="h-48 w-full object-cover" />
			<div class="p-4">
				<p class="mb-2 overflow-hidden truncate text-lg font-medium text-gray-800">
					{product.title}
				</p>
				<div class="flex items-center justify-between">
					<p class="text-xl font-bold">${product.price}</p>
					<button
						class="rounded-full bg-sky-600 px-4 py-2 text-white transition-colors duration-300 hover:bg-sky-700"
						onclick={() => {
							cartProducts.push({
								id: crypto.randomUUID(),
								product,
								quantity: 1
							});
						}}
					>
						Add to cart
					</button>
				</div>
			</div>
		</div>
	{/each}
</div>
