<template>
	<v-form @submit.prevent="addDate">
		<v-text-field
			v-model="newDate.title"
			label="Event Title"
			required
		></v-text-field>

		<v-text-field
			v-model="newDate.date"
			label="Date"
			type="date"
			required
		></v-text-field>

		<v-textarea
			v-model="newDate.description"
			label="Description"
			rows="3"
		></v-textarea>

		<v-btn
			color="primary"
			type="submit"
			block
			class="mt-4"
		>
			Add Important Date
		</v-btn>
	</v-form>
</template>

<script setup>
	import { ref } from 'vue'

	const emit = defineEmits(['add-date'])

	const newDate = ref({
		title: '',
		date: '',
		description: '',
	})

	const addDate = () => {
		if (!newDate.value.title || !newDate.value.date) return

		emit('add-date', {
			title: newDate.value.title,
			date: newDate.value.date,
			description: newDate.value.description,
		})

		// Reset form
		newDate.value = {
			title: '',
			date: '',
			description: '',
		}
	}
</script>
