<script setup>
	defineProps({
		users: {
			type: Array,
			required: true,
		},
	})
</script>

<template>
	<div class="user-list">
		<v-alert
			v-if="users.length === 0"
			type="info"
			text="No team members added yet. Add team members to start tracking points."
			class="mb-4"
		></v-alert>

		<v-data-table
			v-else
			:headers="[
				{ title: 'Name', key: 'name' },
				{ title: 'Role', key: 'role' },
				{ title: 'Points', key: 'points' },
			]"
			:items="[...users].sort((a, b) => b.points - a.points)"
			:items-per-page="10"
			hide-default-footer
		>
			<template v-slot:item="{ item }">
				<tr
					:class="{
						'bg-amber-lighten-5':
							item.points === Math.max(...users.map((u) => u.points)),
					}"
				>
					<td>{{ item.name }}</td>
					<td>{{ item.role }}</td>
					<td>
						<v-chip
							:color="getPointsColor(item.points)"
							variant="outlined"
						>
							{{ item.points }}
						</v-chip>
					</td>
				</tr>
			</template>
		</v-data-table>
	</div>
</template>

<script>
	export default {
		methods: {
			getPointsColor(points) {
				if (points >= 100) return 'success'
				if (points >= 50) return 'info'
				if (points >= 20) return 'warning'
				return 'grey'
			},
		},
	}
</script>

<style scoped>
	.user-list {
		width: 100%;
	}
</style>
