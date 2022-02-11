<template>
	<section>
		<header>
			<span>Restaurants</span> <span><i class="fas fa-store"></i></span>
		</header>
		<div class="restaurants" v-if="restaurants.length">
			<restaurant-item
				v-for="restaurant in restaurants"
				:key="restaurant.id"
				:restaurant="restaurant"
				@click="$router.push(`/buyer/restaurant/${restaurant.id}`)"
			/>
		</div>
	</section>
</template>

<script setup lang="ts">
import { db, collection, getDocs } from "@/includes/firebase";
import RestaurantItem from "./ItemRes.vue";
import { ref } from "vue";

interface res {
	contact: number;
	restaurantName: string;
	location: string;
	photoUrl: string;
	id: string;
}

const restaurants = ref<any[]>(["Hey", "Hello", "hi"]);

const resSnap = await getDocs(collection(db, "restaurants"));
resSnap.forEach((document) => {
	const data = document.data();
	const info: res = {
		...data.info,
		id: document.id,
	};
	restaurants.value.unshift(info);
});
</script>

<style scoped>
/* header {
	font-weight: bold;
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.restaurants {
	height: 38vh;
	display: flex;
	gap: 1rem;
	padding: 1rem 0;
	overflow-x: scroll;
} */
</style>
