<script setup lang="ts">
import { RouterView } from "vue-router";

const res = await fetch("https://source.unsplash.com/random/800x1000/?food");
console.log(res.url);
</script>

<template>
	<RouterView />
</template>

<style>
@import "@/assets/base.css";
</style>
