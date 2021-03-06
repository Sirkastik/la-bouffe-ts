<template>
	<div class="restaurant">
		<div class="top">
			<h2>Welcome to {{ displayName }}</h2>
		</div>
		<h1 v-if="!showDetails">
			<span>The Menu</span><span><i class="fas fa-book-open"></i></span>
		</h1>
		<section v-if="!showDetails">
			<div class="menu" v-if="menu.length">
				<div class="category" v-if="breakfast.length">
					<h1>Breakfast</h1>
					<div v-for="item in breakfast" v-bind:key="item.id" class="item">
						<span class="name">{{ item.name }}</span>
						<span class="price">Ksh. {{ item.price }}</span>
						<span class="delete" @click="handleDelete(item)">
							<i class="fas fa-trash"></i>
						</span>
					</div>
				</div>
				<div class="category" v-if="dishes.length">
					<h1>Dishes</h1>
					<div v-for="item in dishes" v-bind:key="item.id" class="item">
						<span class="name">{{ item.name }}</span>
						<span class="price">Ksh. {{ item.price }}</span>
						<span class="delete" @click="handleDelete(item)">
							<i class="fas fa-trash"></i>
						</span>
					</div>
				</div>
				<div class="category" v-if="snacks.length">
					<h1>Snacks</h1>
					<div v-for="item in snacks" v-bind:key="item.id" class="item">
						<span class="name">{{ item.name }}</span>
						<span class="price">Ksh. {{ item.price }}</span>
						<span class="delete" @click="handleDelete(item)">
							<i class="fas fa-trash"></i>
						</span>
					</div>
				</div>
				<div class="category" v-if="desserts.length">
					<h1>Desserts</h1>
					<div v-for="item in desserts" v-bind:key="item.id" class="item">
						<span class="name">{{ item.name }}</span>
						<span class="price">Ksh. {{ item.price }}</span>
						<span class="delete" @click="handleDelete(item)">
							<i class="fas fa-trash"></i>
						</span>
					</div>
				</div>
				<div class="category" v-if="drinks.length">
					<h1>Drinks</h1>
					<div v-for="item in drinks" v-bind:key="item.id" class="item">
						<span class="name">{{ item.name }}</span>
						<span class="price">Ksh. {{ item.price }}</span>
						<span class="delete" @click="handleDelete(item)">
							<i class="fas fa-trash"></i>
						</span>
					</div>
				</div>
			</div>
			<div class="no-items" v-else @click="showModal = true">
				<span>
					You have no items in your menu. Click here to add food items
				</span>
				<span><i class="fas fa-plus-circle"></i></span>
			</div>
		</section>
		<section v-if="showDetails">
			<div class="info">
				<h3>Restaurant Info</h3>
				<div class="field">
					<span>Name:</span>
					<span contenteditable="true" @blur="edit">
						{{ resDetails.info.restaurantName }}
					</span>
				</div>
				<div class="field">
					<span>Contact:</span>
					<span contenteditable="true" @blur="edit">{{
						resDetails.info.contact
					}}</span>
				</div>
				<div class="field">
					<span>Location:</span>
					<span contenteditable="true" @blur="edit">{{
						resDetails.info.location
					}}</span>
				</div>
			</div>

			<div class="payment">
				<h3>Payment Details</h3>
				<div class="field">
					<span>Payment Name:</span>
					<span contenteditable="true" @blur="edit">{{
						resDetails.payment.paymentName
					}}</span>
				</div>
				<div class="field">
					<span>Payment Type:</span>
					<span contenteditable="true" @blur="edit">{{
						resDetails.payment.paymentType
					}}</span>
				</div>
				<div class="field" v-if="type == 'Buy Goods...'">
					<span>Till Number:</span>
					<span contenteditable="true" @blur="edit">{{
						resDetails.payment.tillNumber
					}}</span>
				</div>
				<div v-else-if="type == 'Pay Bill'">
					<div class="field">
						<span>Business Number:</span>

						<span contenteditable="true" @blur="edit">{{
							resDetails.payment.businessNumber
						}}</span>
					</div>
					<div class="field">
						<span>Account Number:</span>
						<span contenteditable="true" @blur="edit">{{
							resDetails.payment.accNumber
						}}</span>
					</div>
				</div>
				<div class="field" v-else>
					<span>Phone Number:</span>
					<span contenteditable="true" @blur="edit">{{
						resDetails.payment.phoneNumber
					}}</span>
				</div>
				<h3 @click="handleSignout">Sign out</h3>
			</div>
		</section>
	</div>
	<add-modal @close="showModal = false" @add="handleSubmit" v-if="showModal" />
	<app-bottom
		@home="showDetails = false"
		@open="showModal = true"
		@user="getDetails"
	/>
</template>

<script setup lang="ts">
import {
	auth,
	db,
	doc,
	collection,
	getDoc,
	getDocs,
	addDoc,
	deleteDoc,
	query,
	where,
} from "@/includes/firebase";
import AddModal from "@/components/modals/AddModal.vue";
import AppBottom from "@/components/AppBottom.vue";
import { computed, ref } from "vue";
import { useRouter } from "vue-router";

const menu = ref<any[]>([]);
const resDetails = ref<any>({});
const showModal = ref<boolean>(false);
const showDetails = ref<boolean>(false);
const editable = ref<boolean>(false);

const displayName = computed(() => auth.currentUser?.displayName);
const type = computed(() => resDetails.payment.paymentType);
const uid = computed(() => auth.currentUser!.uid);
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

const getItems = (document: any) => {
	const item = {
		...document.data(),
		id: document.id,
	};
	menu.value.unshift(item);
};

const edit = (e: any) => {
	console.log(e);
};

const getMenu = async () => {
	const menu = await Promise.all([
		getDocs(query(collection(db, "dishes"), where("uid", "==", uid.value))),
		getDocs(query(collection(db, "breakfast"), where("uid", "==", uid.value))),
		getDocs(query(collection(db, "drinks"), where("uid", "==", uid.value))),
		getDocs(query(collection(db, "snacks"), where("uid", "==", uid.value))),
		getDocs(query(collection(db, "desserts"), where("uid", "==", uid.value))),
	]);
	menu.forEach((query) => query.forEach(getItems));
};

const handleSubmit = async (data: any) => {
	try {
		const newEntryRef = await addDoc(collection(db, data.category), data);
		const newEntrySnapshot = await getDoc(newEntryRef);
		getItems(newEntrySnapshot);
		console.log("Item Added...");
		showModal.value = false;
	} catch (error) {
		console.log("Couldn't add item...");
		console.log(error);
	}
};

const handleSignout = async () => {
	// await $store.dispatch("signOut");
	console.log("signout");

	useRouter().push("/");
};

const handleDelete = async (itemToDel: any) => {
	try {
		await deleteDoc(doc(db, itemToDel.category, itemToDel.id));
		menu.value = menu.value.filter((item) => item !== itemToDel);
		console.log("Item deleted...");
	} catch (error) {
		console.log("Error deleting item...");
		console.log(error);
	}
};

const getDetails = async () => {
	const resSnap = await getDoc(doc(db, "restaurants", uid.value));
	resDetails.value = resSnap.data();
	console.log(resDetails);
	showDetails.value = true;
};

await getMenu();
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.restaurant {
	text-align: center;
	/* padding: 1rem; */
	min-height: 97vh;
	position: relative;
}

.top {
	padding: 0.9rem;
	margin: 1rem;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
	text-transform: uppercase;
	font-size: 0.9em;
	letter-spacing: 1px;
	font-weight: bold;
}

.restaurant > h1 {
	display: flex;
	justify-content: space-between;
	margin: 0 1.5rem;
	padding: 0.5rem;
	font-size: 1.2em;
	text-transform: uppercase;
	letter-spacing: 1px;
	font-weight: bolder;
	color: var(--lighter);
	border-bottom: 1px solid var(--lighter);
}

section {
	height: calc(100vh - 12.5rem);
	overflow: scroll;
	padding: 1rem 1.5rem;
}

.no-items {
	height: 100%;
	display: grid;
	place-content: center;
	text-transform: uppercase;
	letter-spacing: 1px;
	font-weight: bolder;
	font-size: 0.8em;
	color: var(--lighter);
}

.category {
	margin-bottom: 1rem;
	padding: 1rem;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
}

.category h1 {
	text-transform: uppercase;
	font-size: 0.9em;
	letter-spacing: 1px;
	font-weight: bold;
}

.item {
	border-bottom: 1px solid var(--lighter);
	color: var(--lighter);
	text-transform: uppercase;
	font-size: 0.75em;
	letter-spacing: 1px;
	font-weight: bold;
	display: flex;
	gap: 1rem;
	/* justify-content: space-between; */
	align-items: center;
	margin: 1rem 0;
}

.item span:nth-child(1) {
	flex: 1;
	text-align: left;
	padding-left: 1rem;
}

.item span:nth-child(3) {
	padding: 0.5rem;
	background: var(--main);
	color: var(--bg);
}

.del {
	font-weight: bold;
}

.info,
.payment {
	display: flex;
	flex-direction: column;
	gap: 0.7rem;

	margin: 1rem 0;
}

.field {
	/* border: 1px solid var(--main); */
	display: flex;
	justify-content: space-between;
	gap: 0.5rem;
	background: var(--main);
	color: var(--color);
	padding: 0.5rem 0.8rem;
	border-radius: 0.4rem;
}

.field span:nth-child(2) {
	flex: 1;
}

/* .field > span {
	width: 100%;
} */
</style>
