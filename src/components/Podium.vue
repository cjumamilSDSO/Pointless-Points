<script setup>
	defineProps({
		users: {
			type: Array,
			required: true,
		},
	})

	const getTopThree = (users) => {
		return [...users].sort((a, b) => b.points - a.points).slice(0, 3)
	}
</script>

<template>
	<div class="podium-container">
		<div
			v-if="users.length >= 3"
			class="podium"
		>
			<!-- Second Place -->
			<div class="podium-item">
				<v-avatar
					size="35"
					:color="getTopThree(users)[1].color || 'grey-lighten-1'"
				>
					<v-icon size="32">mdi-numeric-2</v-icon>
				</v-avatar>
				<div
					class="platform"
					:style="{
						height: '120px',
						background: getTopThree(users)[1].color
							? `linear-gradient(180deg, ${getTopThree(users)[1].color}CC 0%, ${
									getTopThree(users)[1].color
							  }66 100%)`
							: 'linear-gradient(180deg, #c0c0c0 0%, #e0e0e0 100%)',
					}"
				>
					<div class="user-info">
						<div class="user-name">{{ getTopThree(users)[1].name }}</div>
						<v-chip color="primary"
							>{{ getTopThree(users)[1].points }} pts</v-chip
						>
					</div>
				</div>
			</div>

			<!-- First Place -->
			<div class="podium-item">
				<v-avatar
					size="64"
					:color="getTopThree(users)[0].color || 'amber'"
				>
					<v-icon size="40">mdi-crown</v-icon>
				</v-avatar>
				<div
					class="platform"
					:style="{
						height: '160px',
						background: getTopThree(users)[0].color
							? `linear-gradient(180deg, ${getTopThree(users)[0].color}CC 0%, ${
									getTopThree(users)[0].color
							  }66 100%)`
							: 'linear-gradient(180deg, #ffd700 0%, #ffed4a 100%)',
					}"
				>
					<div class="user-info">
						<div class="user-name">{{ getTopThree(users)[0].name }}</div>
						<v-chip color="primary"
							>{{ getTopThree(users)[0].points }} pts</v-chip
						>
					</div>
				</div>
			</div>

			<!-- Third Place -->
			<div class="podium-item">
				<v-avatar
					size="30"
					:color="getTopThree(users)[2].color || 'brown-lighten-1'"
				>
					<v-icon size="28">mdi-numeric-3</v-icon>
				</v-avatar>
				<div
					class="platform"
					:style="{
						height: '80px',
						background: getTopThree(users)[2].color
							? `linear-gradient(180deg, ${getTopThree(users)[2].color}CC 0%, ${
									getTopThree(users)[2].color
							  }66 100%)`
							: 'linear-gradient(180deg, #cd7f32 0%, #dea47e 100%)',
					}"
				>
					<div class="user-info">
						<div class="user-name">{{ getTopThree(users)[2].name }}</div>
						<v-chip color="primary"
							>{{ getTopThree(users)[2].points }} pts</v-chip
						>
					</div>
				</div>
			</div>
		</div>
		<v-alert
			v-else
			type="info"
			text="Add at least 3 team members to see the podium!"
			class="mt-4"
		></v-alert>
	</div>
</template>

<style scoped>
	.podium-container {
		display: flex;
		justify-content: center;
		align-items: flex-end;
		min-height: 200px;
		padding: 20px;
	}

	.podium {
		display: flex;
		align-items: flex-end;
		gap: 10px;
	}

	.podium-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 8px;
	}

	.platform {
		width: 100px;
		border-radius: 8px 8px 0 0;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.user-info {
		text-align: center;
		padding: 8px;
	}

	.user-name {
		font-weight: bold;
		margin-bottom: 4px;
		word-break: break-word;
	}
</style>
