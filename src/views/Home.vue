<template>
	<app-header />
	<main>
		<restaurants-section />
		<categories-section />
		<top-picks-section />
	</main>
	<bottom-nav />
</template>

<script setup lang="ts">
import AppHeader from "@/components/AppHeader.vue";
import BottomNav from "@/components/AppNav.vue";

import RestaurantsSection from "@/components/SectionRes.vue";
import CategoriesSection from "@/components/SectionCat.vue";
import TopPicksSection from "@/components/SectionPicks.vue";
</script>

<style scoped>
main {
	height: calc(100vh - 10rem);
	margin-top: 5rem;
	padding: 0 1.5rem;
	overflow-y: scroll;
}
</style>
