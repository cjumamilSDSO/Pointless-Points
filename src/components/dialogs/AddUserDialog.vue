<template>
	<v-form
		ref="form"
		@submit.prevent="submitForm"
	>
		<v-text-field
			v-model="newUser.name"
			label="Name"
			:rules="[rules.required]"
			variant="outlined"
		></v-text-field>

		<v-text-field
			v-model="newUser.role"
			label="Role"
			:rules="[rules.required]"
			variant="outlined"
		></v-text-field>

		<v-text-field
			v-model="newUser.birthdate"
			label="Birthdate"
			type="date"
			:rules="[rules.required]"
			variant="outlined"
		></v-text-field>

		<v-color-picker
			v-model="newUser.color"
			mode="hex"
			hide-inputs
			class="mb-4"
			width="100%"
			style="width: 1000px"
			show-swatches
			hide-canvas
			hide-sliders
		></v-color-picker>

		<v-alert
			v-if="error"
			type="error"
			text="${error}"
			class="mb-4"
		></v-alert>

		<v-btn
			:color="newUser.color || primary"
			type="submit"
			block
			:loading="loading"
		>
			Add Member
		</v-btn>
	</v-form>
</template>

<script setup>
	import { ref } from 'vue'

	const emit = defineEmits(['add-user'])
	const form = ref(null)
	const error = ref('')
	const loading = ref(false)

	const rules = {
		required: (value) => !!value || 'This field is required',
	}

	const newUser = ref({
		name: '',
		role: '',
		birthdate: '',
		color: '#1976D2',
	})

	const submitForm = async () => {
		error.value = ''
		loading.value = true

		const { valid } = await form.value.validate()

		if (!valid) {
			loading.value = false
			return
		}

		emit('add-user', {
			name: newUser.value.name.trim(),
			role: newUser.value.role.trim(),
			birthdate: newUser.value.birthdate,
			color: newUser.value.color,
		})

		// Reset form
		form.value.reset()
		newUser.value = {
			name: '',
			role: '',
			birthdate: '',
			color: '#1976D2',
		}
		loading.value = false
	}
</script>
