<script setup>
import Responsive from "/components/Responsive.vue";
defineProps({
	el: Object,
});
</script>

<template>
	<Responsive Entry>
		<i :class="el.icon"></i>
		<span en-US>{{ el.name["en-US"] }}</span>
		<span zh-CN>{{ el.name["zh-CN"] }}</span>
	</Responsive>
</template>

<script>
export default {
	methods: {
		navigate() {
			DesktopView.navigate(this.MODULE_ID);
		},
	},
};
</script>

<style scoped>
[Entry] {
	display: flex;
	align-items: center;
	padding: 0;
	margin: -1px 0;
	font-weight: normal;
	font-size: 1.1rem;
	color: var(--gray-dark);
}

i {
	color: var(--gray);
	width: 1.2em;
	font-size: 1.2em;
	margin: 0 0.5em 0 0.8em;
}

span {
	flex-grow: 1;
	padding: 0.8em 0;
	border-bottom: 1px solid var(--gray-brighter);
}
</style>