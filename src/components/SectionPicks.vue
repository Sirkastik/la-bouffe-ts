<template>
	<section>
		<header>
			<span>Today's Picks</span> <span><i class="fas fa-fire"></i></span>
		</header>
		<div class="picks">
			<picked-item v-for="item in picks" v-bind:key="item.id" :item="item" />
		</div>
	</section>
</template>

<script setup lang="ts">
import { db, collection, getDocs } from "../includes/firebase";
import PickedItem from "./ItemPicked.vue";

import { ref } from "vue";

const menu = ref<any[]>([]);
const picks = ref<any[]>([]);
const categories: string[] = [
	"breakfast",
	"dishes",
	"drinks",
	"snacks",
	"desserts",
];
const links: string[] = [
	"https://images.unsplash.com/photo-1502302800694-ff6ec595e6e3?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1000&ixid=MnwxfDB8MXxyYW5kb218MHx8dGVhP2NvZmZlZXx8fHx8fDE2NDIyNDEwMzc&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1000",
	"https://images.unsplash.com/photo-1597393353296-d149e4d4d771?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1000&ixid=MnwxfDB8MXxyYW5kb218MHx8bHVuY2h8fHx8fHwxNjQyMjQxMTEy&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1000",
	"https://images.unsplash.com/photo-1618046364546-81e9d03d39a6?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1000&ixid=MnwxfDB8MXxyYW5kb218MHx8ZHJpbmtzfHx8fHx8MTY0MjI0MTE3MQ&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1000",
	"https://images.unsplash.com/photo-1608219994488-cc269412b3e4?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1000&ixid=MnwxfDB8MXxyYW5kb218MHx8ZnJpZXN8fHx8fHwxNjQyMjQwOTA4&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1000",
	"https://images.unsplash.com/photo-1506095619733-3c3ea98fb968?crop=entropy&cs=tinysrgb&fit=crop&fm=jpg&h=1000&ixid=MnwxfDB8MXxyYW5kb218MHx8ZGVzc2VydHx8fHx8fDE2NDIyNDEyMzE&ixlib=rb-1.2.1&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1000",
];

const getMenu = async () => {
	const menu = await Promise.all([
		getDocs(collection(db, "dishes")),
		getDocs(collection(db, "breakfast")),
		getDocs(collection(db, "drinks")),
		getDocs(collection(db, "snacks")),
		getDocs(collection(db, "desserts")),
	]);
	menu.forEach((query) => query.forEach(getItems));
};

const randomPick = (arr: any[]) => {
	return arr[Math.floor(Math.random() * arr.length)];
};

const getItems = (document: any) => {
	const item = {
		...document.data(),
		id: document.id,
	};
	menu.value.unshift(item);
};

await getMenu();
categories.forEach((category, index) => {
	let pick = randomPick(
		menu.value.filter((item) => item.category === `${category}`)
	);
	pick.photoUrl = links[index];
	pick.category = category;
	picks.value.push(pick);
});
</script>

<style scoped>
header {
	font-weight: bold;
	display: flex;
	justify-content: space-between;
	align-items: center;
}

.picks {
	height: 30vh;
	display: flex;
	gap: 1rem;
	padding: 1rem 0;
	overflow-x: scroll;
}
</style>
