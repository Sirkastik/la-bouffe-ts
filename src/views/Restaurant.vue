<template>
	<div class="restPage">
		<div v-if="info" class="img">
			<img :src="info.photoUrl" alt="" />
		</div>
		<h3 v-if="info" class="title">{{ info.restaurantName }}</h3>
		<!-- <span class="back"><i class="fas fa-arrow-left"></i></span> -->
		<div class="back" @click="$router.push('/')">
			<span class="down" style="--size: 2">
				<i class="fas fa-chevron-down"></i>
			</span>
			<span class="down" style="--size: 3">
				<i class="fas fa-chevron-down"></i>
			</span>
		</div>
		<div class="wrapper">
			<h1>
				<span @click="prev"><i class="fas fa-chevron-left"></i></span>
				<div class="name">{{ listName }}</div>
				<span @click="next"><i class="fas fa-chevron-right"></i></span>
			</h1>
			<div class="list contents" v-if="listName === 'breakfast'">
				<div
					class="breakfast list-item"
					v-for="item in breakfast"
					:key="item.id"
				>
					<span class="icon"><i class="fas fa-coffee"></i></span>
					<span class="item-name">{{ item.name }}</span>
					<span class="item-price">Ksh.{{ item.price }}</span>
					<span class="atc" @click="addToCart(item)">
						<i class="fas fa-cart-plus"></i>
					</span>
				</div>
			</div>
			<div class="list contents" v-if="listName === 'dishes'">
				<div class="dishes list-item" v-for="item in dishes" :key="item.id">
					<span class="icon"><i class="fas fa-utensils"></i></span>
					<span class="item-name">{{ item.name }}</span>
					<span class="item-price">Ksh.{{ item.price }}</span>
					<span class="atc" @click="addToCart(item)">
						<i class="fas fa-cart-plus"></i>
					</span>
				</div>
			</div>
			<div class="list contents" v-if="listName === 'drinks'">
				<div class="drinks list-item" v-for="item in drinks" :key="item.id">
					<span class="icon"><i class="fas fa-cocktail"></i></span>
					<span class="item-name">{{ item.name }}</span>
					<span class="item-price">Ksh.{{ item.price }}</span>
					<span class="atc" @click="addToCart(item)">
						<i class="fas fa-cart-plus"></i>
					</span>
				</div>
			</div>
			<div class="list contents" v-if="listName === 'desserts'">
				<div class="desserts list-item" v-for="item in desserts" :key="item.id">
					<span class="icon"><i class="fas fa-cookie"></i></span>
					<span class="item-name">{{ item.name }}</span>
					<span class="item-price">Ksh.{{ item.price }}</span>
					<span class="atc" @click="addToCart(item)">
						<i class="fas fa-cart-plus"></i>
					</span>
				</div>
			</div>
			<div class="list contents" v-if="listName === 'snacks'">
				<div class="snacks list-item" v-for="item in snacks" :key="item.id">
					<span class="icon"><i class="fas fa-hamburger"></i></span>
					<span class="item-name">{{ item.name }}</span>
					<span class="item-price">Ksh.{{ item.price }}</span>
					<span class="atc" @click="addToCart(item)">
						<i class="fas fa-cart-plus"></i>
					</span>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
import { db, collection, getDocs, query, where } from "@/includes/firebase";
import { computed, ref } from "vue";
import { useRoute } from "vue-router";

const menu = ref<any[]>([]);
const resDetails = ref<{ info: any; payment: any }>({ info: {}, payment: {} });
const categories: string[] = [
	"breakfast",
	"dishes",
	"drinks",
	"snacks",
	"desserts",
];
const chosen = ref<number>(0);
const cart = ref<any[]>([]);

const id = computed(() => useRoute().params.id.toString());
const info = computed(() => resDetails.value.info);
const listName = computed(() => tabList.value[chosen.value]);
const tabList = computed(() => {
	let list: string[] = [];
	categories.forEach((category) => {
		let array = menu.value.filter((item) => item.category === `${category}`);
		if (array.length) list.push(category);
	});
	return list;
});
const breakfast = computed(() =>
	menu.value.filter((item) => item.category === "breakfast")
);
const dishes = computed(() =>
	menu.value.filter((item) => item.category === "dishes")
);
const snacks = computed(() =>
	menu.value.filter((item) => item.category === "snacks")
);
const desserts = computed(() =>
	menu.value.filter((item) => item.category === "desserts")
);
const drinks = computed(() =>
	menu.value.filter((item) => item.category === "drinks")
);

const getMenu = async () => {
	const querySnapShots = await Promise.all([
		getDocs(query(collection(db, "dishes"), where("uid", "==", id.value))),
		getDocs(query(collection(db, "breakfast"), where("uid", "==", id.value))),
		getDocs(query(collection(db, "drinks"), where("uid", "==", id.value))),
		getDocs(query(collection(db, "snacks"), where("uid", "==", id.value))),
		getDocs(query(collection(db, "desserts"), where("uid", "==", id.value))),
	]);
	querySnapShots.forEach((querySnapShot) => querySnapShot.forEach((document) => {
	const item = {
		...document.data(),
		id: document.id,
	};
	menu.value.unshift(item);
}
));
};

const next = () => {
	const max = tabList.value.length - 1;
	if (chosen.value === max) chosen.value = 0;
	else chosen.value += 1;
};
const prev = () => {
	if (chosen.value === 0) chosen.value = tabList.value.length - 1;
	else chosen.value -= 1;
};

const addToCart = (item: any) => {
	const myCart = cart.value ? addItem(item) : [item];
	localStorage.setItem("foodCart", JSON.stringify(myCart));
};

const addItem = (item: any) => {
	let myCart = cart.value;
	if (inCart(item)) return myCart;
	myCart.push(item);
	return myCart;
};

const inCart = (newItem: any) => {
	return cart.value && cart.value.some((item) => item.id === newItem.id);
};

await getMenu();
const savedCart = localStorage.getItem("foodCart");
if (savedCart) cart.value = JSON.parse(savedCart);
</script>

<style scoped>
.restPage {
	text-align: center;
	min-height: 100vh;
}

.img {
	height: 30vh;
	border-radius: 0 0 2rem 2rem;
	overflow: hidden;
}

img {
	object-position: 0 -20vh;
	width: 100%;
	aspect-ratio: 4/5;
	filter: brightness(80%);
}

.title {
	position: absolute;
	inset: 26vh 0 auto 0;
	width: max-content;
	z-index: 10;
	margin: auto;
	padding: 1rem 2.5rem;
	border-radius: 0.5rem;

	font-size: 0.9em;
	text-transform: uppercase;
	letter-spacing: 1px;
	font-weight: bold;
	box-shadow: var(--shadow);
	color: var(--main);
}

.title::before {
	content: "";
	position: absolute;
	inset: 0;
	z-index: -1;
	background: var(--grad);
	background-size: 500%;
	background-position: center;
	opacity: 0.97;
	border-radius: inherit;
}

.back {
	position: absolute;
	inset: 1rem auto auto 1rem;
	z-index: 5;
	display: flex;
	flex-direction: column;
	align-items: center;
	color: var(--main);
	padding: 0.5rem;
	border-radius: 1rem;
}

.back::before {
	content: "";
	position: absolute;
	inset: 0;
	z-index: -1;
	background: var(--grad);
	background-size: 500%;
	background-position: center;
	opacity: 0.9;
	border-radius: inherit;
}

.down {
	font-size: calc(0.2rem * var(--size));
}

h1 {
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding: 0 0.5rem;
}

h1 > span {
	height: 1.5rem;
	width: 1.5rem;
	display: grid;
	place-content: center;
	font-size: 0.75rem;
	border-radius: 50%;
	background: var(--color);
	color: var(--main);
	box-shadow: var(--shadow);
}

.name {
	color: var(--lighter);
	display: inline-block;
	font-size: 1em;
	text-transform: uppercase;
	letter-spacing: 1px;
	font-weight: bold;
}

.wrapper {
	margin-top: 1.5rem;
	padding: 1rem 1rem;
}

.list {
	display: flex;
	flex-direction: column;
	gap: 1rem;
	padding: 1rem 0;
	width: 100%;
	animation: slide 1s ease-in-out both;
}

@keyframes slide {
	from {
		transform: translateX(70vh);
	}
	to {
		transform: translateX(0);
	}
}

.list-item {
	display: flex;
	justify-content: space-between;
	align-items: center;
	box-shadow: var(--shadow);
	height: 3rem;
	border-radius: 0.25rem;

	color: var(--lighter);
	font-size: 0.8em;
	text-transform: uppercase;
	letter-spacing: 1px;
	font-weight: bold;
}

.icon {
	height: 3rem;
	background: var(--color);
	color: var(--main);
	padding: 0 0.5rem;
	display: grid;
	place-content: center;
	border-radius: 0.25rem 0 0 0.25rem;
}

.item-name {
	margin-right: auto;
	padding-left: 1rem;
}

.item-price {
	flex-shrink: 1;
	padding-right: 1rem;
}

.atc {
	height: 3rem;
	background: var(--color);
	color: var(--main);
	padding: 0 0.5rem;
	display: grid;
	place-content: center;
	border-radius: 0 0.25rem 0.25rem 0;
}
</style>
