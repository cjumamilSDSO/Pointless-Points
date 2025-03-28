<template>
	<v-form
		ref="form"
		@submit.prevent="addImportantDate"
	>
		<v-text-field
			v-model="title"
			label="Event Title"
			:rules="[rules.required]"
			variant="outlined"
			clearable
		></v-text-field>

		<v-text-field
			v-model="date"
			label="Date"
			type="date"
			:rules="[rules.required]"
			variant="outlined"
		></v-text-field>

		<v-select
			v-model="color"
			:items="colorOptions"
			label="Event Color"
			variant="outlined"
		></v-select>

		<v-btn
			type="submit"
			color="primary"
			block
			:loading="false"
		>
			Add Important Date
		</v-btn>
	</v-form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['add-date'])

const title = ref('')
const date = ref('')
const color = ref('error')
const form = ref(null)

const colorOptions = [
	{ text: 'Red', value: 'error' },
	{ text: 'Green', value: 'success' },
	{ text: 'Blue', value: 'info' },
	{ text: 'Orange', value: 'warning' },
]

const rules = {
	required: (value) => !!value || 'Required field',
}

const addImportantDate = async () => {
	const { valid } = await form.value.validate()

	if (!valid) return

	emit('add-date', {
		title: title.value.trim(),
		date: date.value,
		color: color.value,
	})

	title.value = ''
	date.value = ''
	form.value.reset()
}
</script>
