<script setup>
	import { ref, computed } from "vue"

	const props = defineProps({
		users: {
			type: Array,
			required: true,
		},
		importantDates: {
			type: Array,
			required: true,
			default: () => [],
		},
	})

	const currentDate = new Date()
	const currentMonth = ref(currentDate.getMonth())
	const currentYear = ref(currentDate.getFullYear())

	const months = [
		"January",
		"February",
		"March",
		"April",
		"May",
		"June",
		"July",
		"August",
		"September",
		"October",
		"November",
		"December",
	]

	const daysInMonth = computed(() => {
		return new Date(currentYear.value, currentMonth.value + 1, 0).getDate()
	})

	const firstDayOfMonth = computed(() => {
		return new Date(currentYear.value, currentMonth.value, 1).getDay()
	})

	const calendarDays = computed(() => {
		const days = []
		const totalDays = daysInMonth.value
		const firstDay = firstDayOfMonth.value

		for (let i = 0; i < firstDay; i++) {
			days.push({ day: "", events: [] })
		}

		for (let day = 1; day <= totalDays; day++) {
			const events = []

			props.users.forEach((user) => {
				if (user.birthdate) {
					const bDate = new Date(user.birthdate)
					if (
						bDate.getMonth() === currentMonth.value &&
						bDate.getDate() === day
					) {
						events.push({
							type: "birthday",
							title: `${user.name}'s Birthday`,
							color: "primary",
						})
					}
				}
			})

			props.importantDates.forEach((date) => {
				const iDate = new Date(date.date)
				if (
					iDate.getMonth() === currentMonth.value &&
					iDate.getDate() === day
				) {
					events.push({
						type: "important",
						title: date.title,
						color: date.color || "error",
					})
				}
			})

			days.push({ day, events })
		}

		return days
	})

	const nextMonth = () => {
		if (currentMonth.value === 11) {
			currentMonth.value = 0
			currentYear.value++
		} else {
			currentMonth.value++
		}
	}

	const prevMonth = () => {
		if (currentMonth.value === 0) {
			currentMonth.value = 11
			currentYear.value--
		} else {
			currentMonth.value--
		}
	}
</script>

<template>
	<div class="calendar">
		<div class="calendar-header">
			<v-btn
				icon="mdi-chevron-left"
				@click="prevMonth"
			></v-btn>
			<h3>{{ months[currentMonth] }} {{ currentYear }}</h3>
			<v-btn
				icon="mdi-chevron-right"
				@click="nextMonth"
			></v-btn>
		</div>

		<div class="calendar-grid">
			<div class="calendar-day-header">Sun</div>
			<div class="calendar-day-header">Mon</div>
			<div class="calendar-day-header">Tue</div>
			<div class="calendar-day-header">Wed</div>
			<div class="calendar-day-header">Thu</div>
			<div class="calendar-day-header">Fri</div>
			<div class="calendar-day-header">Sat</div>

			<div
				v-for="(dayData, index) in calendarDays"
				:key="index"
				class="calendar-day"
				:class="{ empty: !dayData.day }"
			>
				<div class="day-number">{{ dayData.day }}</div>
				<div class="day-events">
					<v-chip
						v-for="(event, eventIndex) in dayData.events"
						:key="eventIndex"
						:color="event.color"
						size="x-small"
						class="mt-1"
						:prepend-icon="
							event.type === 'birthday' ? 'mdi-cake-variant' : 'mdi-star'
						"
					>
						{{ event.title }}
					</v-chip>
				</div>
			</div>
		</div>
	</div>
</template>

<style scoped>
	.calendar {
		border: 1px solid var(--v-border-color);
		border-radius: 8px;
		padding: 16px;
	}

	.calendar-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 16px;
	}

	.calendar-grid {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		gap: 4px;
	}

	.calendar-day-header {
		text-align: center;
		font-weight: bold;
		padding: 8px;
		background-color: var(--v-theme-surface-variant);
		border-radius: 4px;
	}

	.calendar-day {
		min-height: 80px;
		padding: 4px;
		border: 1px solid var(--v-border-color);
		border-radius: 4px;
	}

	.calendar-day.empty {
		background-color: var(--v-theme-surface-variant);
	}

	.day-number {
		font-weight: bold;
		margin-bottom: 4px;
	}

	.day-events {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	@media (max-width: 960px) {
		.calendar-day {
			min-height: 60px;
		}
	}
</style>
