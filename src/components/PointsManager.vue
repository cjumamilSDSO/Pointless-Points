<template>
	<v-form
		ref="form"
		@submit.prevent="awardPoints"
	>
		<v-select
			v-model="selectedUserId"
			:items="users"
			item-title="name"
			item-value="id"
			label="Select Team Member"
			:rules="[rules.required]"
			variant="outlined"
			:disabled="users.length === 0"
			class="mb-4"
		>
			<template v-slot:item="{ props, item }">
				<v-list-item v-bind="props">
					{{ item.raw.name }} ({{ item.raw.role }})
				</v-list-item>
			</template>
		</v-select>

		<v-row
			align="center"
			class="mb-4"
		>
			<v-col
				cols="12"
				sm="8"
			>
				<v-text-field
					v-model.number="pointsToAdd"
					type="number"
					label="Points to Award"
					:rules="[rules.required, rules.positive]"
					variant="outlined"
					:disabled="users.length === 0"
					hide-spin-buttons
				>
					<template v-slot:append-inner>
						<div class="d-flex align-center">
							<v-btn
								icon="mdi-minus"
								variant="text"
								size="small"
								@click="pointsToAdd > 1 ? pointsToAdd-- : null"
								:disabled="pointsToAdd <= 1"
							></v-btn>
							<v-btn
								icon="mdi-plus"
								variant="text"
								size="small"
								@click="pointsToAdd++"
							></v-btn>
						</div>
					</template>
				</v-text-field>
			</v-col>
		</v-row>

		<v-alert
			v-if="message"
			:type="message.includes('Awarded') ? 'success' : 'error'"
			:text="message"
			class="mb-4"
		></v-alert>

		<v-btn
			type="submit"
			color="primary"
			block
			:disabled="users.length === 0"
			:loading="false"
		>
			Award Points
		</v-btn>
	</v-form>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
	users: {
		type: Array,
		required: true,
	},
})

const emit = defineEmits(['add-points'])

const selectedUserId = ref('')
const pointsToAdd = ref(1)
const message = ref('')
const form = ref(null)

const rules = {
	required: (value) => !!value || 'Required field',
	positive: (value) => value > 0 || 'Points must be positive',
}

const awardPoints = async () => {
	const { valid } = await form.value.validate()

	if (!valid) return

	// Emit event to parent component
	emit('add-points', parseInt(selectedUserId.value), pointsToAdd.value)

	// Show success message
	const user = props.users.find((u) => u.id === parseInt(selectedUserId.value))
	message.value = `Awarded ${pointsToAdd.value} point${
		pointsToAdd.value > 1 ? 's' : ''
	} to ${user.name}`

	// Reset form after delay
	setTimeout(() => {
		message.value = ''
	}, 3000)
}
</script>

<style scoped>
/* Vuetify handles styling */
</style>
