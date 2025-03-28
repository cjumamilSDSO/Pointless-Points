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

		<v-select
			v-model="color"
			:items="colorOptions"
			label="Member Color"
			:rules="[rules.required]"
			variant="outlined"
			class="mb-4"
			item-title="title"
			item-value="value"
		>
			<template v-slot:selection="{ item }">
				<v-avatar
					size="24"
					class="mr-2"
					:style="{ backgroundColor: item.value }"
				></v-avatar>
				{{ item.title }}
			</template>
			<template v-slot:item="{ props, item }">
				<v-list-item v-bind="props">
					<template v-slot:prepend>
						<v-avatar
							size="24"
							class="mr-2"
							:style="{ backgroundColor: item.raw.value }"
						></v-avatar>
					</template>
				</v-list-item>
			</template>
		</v-select>

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
const color = ref('')
const form = ref(null)

const colorOptions = [
	{ title: 'Blue', value: '#1976D2' },
	{ title: 'Purple', value: '#9C27B0' },
	{ title: 'Green', value: '#4CAF50' },
	{ title: 'Orange', value: '#FF9800' },
	{ title: 'Red', value: '#F44336' },
	{ title: 'Teal', value: '#009688' },
]

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
		color: color.value,
	})

	name.value = ''
	role.value = ''
	birthdate.value = ''
	color.value = ''
	error.value = ''
	form.value.reset()
}
</script>
