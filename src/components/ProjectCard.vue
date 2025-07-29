<template>
	<div class="p-4 border h-100">
	    <img :src="project.image" width="250" alt="">

	    <h3>{{ project.title }}</h3>
	    <p>{{ project.description }}</p>
	</div>
</template>

<script setup>
	import {defineProps} from 'vue';

	defineProps({
		project: Object
	})
</script>