<template>
	<v-app>
		<v-app-bar color="primary">
			<div class="d-flex align-center mr-4">
				<v-toolbar-title class="ml-5">
					<span>Sia's Pointless Points</span>
				</v-toolbar-title>
				<img
					src="@/assets/Official Sia Coin.png"
					style="max-height: 35px; margin-left: 8px"
				/>
			</div>

			<v-spacer />

			<v-btn
				@click="toggleSupervisor"
				:color="isSupervisor ? 'error' : 'success'"
				variant="tonal"
			>
				{{ isSupervisor ? 'Exit Supervisor Mode' : 'Supervisor Mode' }}
			</v-btn>
		</v-app-bar>

		<v-main>
			<v-container>
				<v-row>
					<v-col
						cols="12"
						md="8"
					>
						<v-card class="mb-4">
							<v-card-title>Team Calendar</v-card-title>
							<v-card-text>
								<Calendar
									:users="users"
									:important-dates="importantDates"
								/>
							</v-card-text>
						</v-card>

						<v-card class="mb-4">
							<v-card-title>Winners Podium</v-card-title>
							<v-card-text>
								<Podium :users="users" />
							</v-card-text>
						</v-card>

						<v-card class="mb-4">
							<v-card-title>Points Distribution</v-card-title>
							<v-card-text>
								<BarGraph :users="users" />
							</v-card-text>
						</v-card>

						<v-card class="mb-4">
							<v-card-title>Team Members</v-card-title>
							<v-card-text>
								<UserList :users="users" />
							</v-card-text>
						</v-card>

						<v-card>
							<v-card-title>Activity Log</v-card-title>
							<v-card-text>
								<AuditLog :logs="auditLogs" />
							</v-card-text>
						</v-card>
					</v-col>

					<v-col
						cols="12"
						md="4"
					>
						<v-card
							v-if="isSupervisor"
							class="mb-4"
						>
							<v-card-title>Add New Member</v-card-title>
							<v-card-text>
								<AddUser @add-user="addUser" />
							</v-card-text>
						</v-card>

						<v-card
							v-if="isSupervisor"
							class="mb-4"
						>
							<v-card-title>Add Important Date</v-card-title>
							<v-card-text>
								<ImportantDates @add-date="addImportantDate" />
							</v-card-text>
						</v-card>

						<v-card v-if="isSupervisor">
							<v-card-title>Manage Points</v-card-title>
							<v-card-text>
								<PointsManager
									:users="users"
									@add-points="addPoints"
								/>
							</v-card-text>
						</v-card>
					</v-col>
				</v-row>
			</v-container>
		</v-main>

		<v-footer
			app
			color="primary"
			class="text-center d-flex justify-center"
		>
			<span class="text-white"
				>Team Roll Call Points App - {{ new Date().getFullYear() }}</span
			>
		</v-footer>
	</v-app>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import UserList from './components/UserList.vue'
import AddUser from './components/AddUser.vue'
import PointsManager from './components/PointsManager.vue'
import AuditLog from './components/AuditLog.vue'
import Podium from './components/Podium.vue'
import Calendar from './components/Calendar.vue'
import ImportantDates from './components/ImportantDates.vue'
import BarGraph from './components/BarGraph.vue'

// Store users in localStorage to persist data
const users = ref([])
const auditLogs = ref([])
const importantDates = ref([])
const isSupervisor = ref(false)

// Load users from localStorage if available
onMounted(() => {
	const savedUsers = localStorage.getItem('roll-call-users')
	const savedLogs = localStorage.getItem('roll-call-audit-logs')
	const savedDates = localStorage.getItem('roll-call-important-dates')

	if (savedUsers) {
		users.value = JSON.parse(savedUsers)
	}
	if (savedLogs) {
		auditLogs.value = JSON.parse(savedLogs)
	}
	if (savedDates) {
		importantDates.value = JSON.parse(savedDates)
	}
})

// Function to add a new user
const addUser = (user) => {
	users.value.push({
		id: Date.now(),
		name: user.name,
		role: user.role,
		birthdate: user.birthdate,
		color: user.color,
		points: 0,
	})
	saveUsers()
	addAuditLog('add', `Added new team member: ${user.name} (${user.role})`)
}

// Function to save users to localStorage
const saveUsers = () => {
	localStorage.setItem('roll-call-users', JSON.stringify(users.value))
}

// Function to add points to a user
const addPoints = (userId, points) => {
	const user = users.value.find((u) => u.id === userId)
	if (user) {
		user.points += points
		saveUsers()
		addAuditLog(
			'points',
			`Awarded ${points} point${points > 1 ? 's' : ''} to ${user.name}`
		)
	}
}

// Function to add an important date
const addImportantDate = (date) => {
	importantDates.value.push({
		id: Date.now(),
		...date,
	})
	saveImportantDates()
	addAuditLog('date', `Added important date: ${date.title}`)
}

// Function to save important dates to localStorage
const saveImportantDates = () => {
	localStorage.setItem(
		'roll-call-important-dates',
		JSON.stringify(importantDates.value)
	)
}

// Function to add an audit log
const addAuditLog = (action, message) => {
	const log = {
		action,
		message,
		timestamp: Date.now(),
	}
	auditLogs.value.unshift(log)
	localStorage.setItem('roll-call-audit-logs', JSON.stringify(auditLogs.value))
}

// Toggle supervisor mode
const toggleSupervisor = () => {
	isSupervisor.value = !isSupervisor.value
}
</script>

<style>
/* Remove existing styles as they're handled by Vuetify */
</style>
