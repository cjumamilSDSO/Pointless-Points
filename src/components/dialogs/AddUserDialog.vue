<template>
	<v-form @submit.prevent="addUser">
		<v-text-field
			v-model="newUser.name"
			label="Name"
			required
		></v-text-field>

		<v-text-field
			v-model="newUser.role"
			label="Role"
			required
		></v-text-field>

		<v-text-field
			v-model="newUser.birthdate"
			label="Birthdate"
			type="date"
			required
		></v-text-field>

		<v-color-picker
			v-model="newUser.color"
			mode="hex"
			hide-inputs
			class="mx-auto mb-4"
		></v-color-picker>

		<v-btn
			color="primary"
			type="submit"
			block
		>
			Add Member
		</v-btn>
	</v-form>
</template>

<script setup>
	import { ref } from 'vue'

	const emit = defineEmits(['add-user'])

	const newUser = ref({
		name: '',
		role: '',
		birthdate: '',
		color: '#1976D2',
	})

	const addUser = () => {
		if (!newUser.value.name || !newUser.value.role || !newUser.value.birthdate)
			return

		emit('add-user', {
			name: newUser.value.name,
			role: newUser.value.role,
			birthdate: newUser.value.birthdate,
			color: newUser.value.color,
		})

		// Reset form
		newUser.value = {
			name: '',
			role: '',
			birthdate: '',
			color: '#1976D2',
		}
	}
</script>
