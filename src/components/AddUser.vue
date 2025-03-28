<template>
	<v-form
		ref="form"
		@submit.prevent="addUser"
	>
		<v-text-field
			v-model="name"
			label="Name"
			:rules="[rules.required]"
			variant="outlined"
			clearable
		></v-text-field>

		<v-text-field
			v-model="role"
			label="Role"
			:rules="[rules.required]"
			variant="outlined"
			clearable
		></v-text-field>

		<v-text-field
			v-model="birthdate"
			label="Birthdate"
			type="date"
			:rules="[rules.required]"
			variant="outlined"
		></v-text-field>

		<v-alert
			v-if="error"
			type="error"
			:text="error"
			class="mb-4"
		></v-alert>

		<v-btn
			type="submit"
			color="primary"
			block
			:loading="false"
		>
			Add Team Member
		</v-btn>
	</v-form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['add-user'])

const name = ref('')
const role = ref('')
const birthdate = ref('')
const error = ref('')
const form = ref(null)

const rules = {
	required: (value) => !!value || 'Required field',
}

const addUser = async () => {
	const { valid } = await form.value.validate()

	if (!valid) return

	emit('add-user', {
		name: name.value.trim(),
		role: role.value.trim(),
		birthdate: birthdate.value,
	})

	name.value = ''
	role.value = ''
	birthdate.value = ''
	error.value = ''
	form.value.reset()
}
</script>
