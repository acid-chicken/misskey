<template>
<div class="cpu">
	<x-pie class="pie" :value="usage"/>
	<div>
		<p>%fa:microchip%CPU</p>
		<p>{{ meta.cpu.cores }} Cores</p>
		<p>{{ meta.cpu.model }}</p>
	</div>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import XPie from './server.pie.vue';

export default Vue.extend({
	components: {
		XPie
	},
	props: ['connection', 'meta'],
	data() {
		return {
			usage: 0
		};
	},
	mounted() {
		this.connection.on('stats', this.onStats);
	},
	beforeDestroy() {
		this.connection.off('stats', this.onStats);
	},
	methods: {
		onStats(stats) {
			this.usage = stats.cpu_usage;
		}
	}
});
</script>

<style lang="stylus" scoped>
root(isDark)
	display grid
	grid 80px / 80px 1fr
	grid-gap 10px
	padding 10px

	> div > p
		margin 0
		font-size 12px
		color isDark ? #a8b4bd : #505050

		&:first-child
			font-weight bold

			> [data-fa]
				margin-right 4px

.cpu[data-darkmode]
	root(true)

.cpu:not([data-darkmode])
	root(false)

</style>
