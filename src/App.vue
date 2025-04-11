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
        {{ isSupervisor ? "Exit Supervisor Mode" : "Supervisor Mode" }}
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col>
            <v-card>
              <v-card-text>
                <strong>Announcements:</strong>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6">
            <v-card class="mb-4">
              <v-card-title class="d-flex"> Top 3 Podium </v-card-title>
              <v-card-text>
                <Podium :users="users" />
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12" md="6">
            <v-card class="mb-4">
              <v-card-title class="d-flex">
                {{ showListView ? "List View" : "Graph View" }}
                <v-spacer></v-spacer>

                <v-btn
                  v-if="showListView && isSupervisor"
                  icon
                  variant="plain"
                  color="primary"
                  @click="showAddUserDialog = true"
                >
                  <v-icon> mdi-account-plus-outline</v-icon>
                </v-btn>

                <v-btn icon variant="plain" color="primary" @click="showList">
                  <v-icon> mdi-list-box-outline</v-icon>
                </v-btn>

                <v-btn
                  icon
                  variant="plain"
                  color="primary"
                  @click="showBarGraphView"
                >
                  <v-icon> mdi-chart-bar</v-icon>
                </v-btn>
              </v-card-title>

              <v-card-text>
                <UserList v-if="showListView" :users="users" />
                <BarGraph v-else :users="users" />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" md="6">
            <v-card class="mb-4">
              <v-card-title class="d-flex">
                Team Calendar
                <v-spacer></v-spacer>

                <v-btn
                  v-if="showListView && isSupervisor"
                  icon
                  variant="plain"
                  color="primary"
                  @click="showImportantDateDialog = true"
                >
                  <v-icon> mdi-plus-circle-outline</v-icon>
                </v-btn>
              </v-card-title>

              <v-card-text>
                <Calendar :users="users" :important-dates="importantDates" />
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12" md="6">
            <v-card class="mb-4">
              <v-card-title>Activity Log</v-card-title>

              <v-card-text>
                <AuditLog :logs="auditLogs" />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>

      <v-dialog v-model="showAddUserDialog" max-width="500px">
        <v-card>
          <v-card-title>Add New Member</v-card-title>

          <v-card-text>
            <AddUser @add-user="addUserAndClose" />
          </v-card-text>

          <v-card-actions class="justify-start">
            <v-btn
              color="error"
              variant="text"
              @click="showAddUserDialog = false"
            >
              Close
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog v-model="showImportantDateDialog" max-width="500px">
        <v-card>
          <v-card-title>Add an Event</v-card-title>

          <v-card-text>
            <ImportantDates @add-date="addDateAndClose" />
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>

            <v-btn
              color="primary"
              variant="text"
              @click="showImportantDateDialog = false"
              >Close</v-btn
            >
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog v-model="showPointsManagerDialog" max-width="500px">
        <v-card>
          <v-card-title>Manage Points</v-card-title>
          <v-card-text>
            <PointsManager :users="users" @add-points="addPointsAndClose" />
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>

            <v-btn
              color="primary"
              variant="text"
              @click="showPointsManagerDialog = false"
            >
              Close
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-main>

    <v-footer app color="primary" class="text-center d-flex justify-center">
      <span class="text-white"
        >Team Roll Call Points App - {{ new Date().getFullYear() }}</span
      >
    </v-footer>
  </v-app>
</template>

<script setup>
import { ref, onMounted } from "vue";
import UserList from "./components/UserList.vue";
import AddUser from "./components/dialogs/AddUserDialog.vue";
import PointsManager from "./components/dialogs/ManagePointsDialog.vue";
import AuditLog from "./components/AuditLog.vue";
import Podium from "./components/Podium.vue";
import Calendar from "./components/Calendar.vue";
import ImportantDates from "./components/dialogs/AddCalendarEventDialog.vue";
import BarGraph from "./components/BarGraph.vue";

// Store users in localStorage to persist data
const users = ref([]);
const auditLogs = ref([]);
const importantDates = ref([]);
const isSupervisor = ref(false);
const showListView = ref(true); // Add this to control which view to show

// Add dialog control refs
const showAddUserDialog = ref(false);
const showImportantDateDialog = ref(false);
const showPointsManagerDialog = ref(false);

// Load users from localStorage if available
onMounted(() => {
  const savedUsers = localStorage.getItem("roll-call-users");
  const savedLogs = localStorage.getItem("roll-call-audit-logs");
  const savedDates = localStorage.getItem("roll-call-important-dates");

  if (savedUsers) {
    users.value = JSON.parse(savedUsers);
  }
  if (savedLogs) {
    auditLogs.value = JSON.parse(savedLogs);
  }
  if (savedDates) {
    importantDates.value = JSON.parse(savedDates);
  }
});

// Function to add a new user
const addUser = (user) => {
  users.value.push({
    id: Date.now(),
    name: user.name,
    role: user.role,
    birthdate: user.birthdate,
    color: user.color,
    points: 0,
  });
  saveUsers();
  addAuditLog("add", `Added new team member: ${user.name} (${user.role})`);
};

// Function to save users to localStorage
const saveUsers = () => {
  localStorage.setItem("roll-call-users", JSON.stringify(users.value));
};

// Function to add points to a user
const addPoints = (userId, points) => {
  const user = users.value.find((u) => u.id === userId);
  if (user) {
    user.points += points;
    saveUsers();
    addAuditLog(
      "points",
      `Awarded ${points} point${points > 1 ? "s" : ""} to ${user.name}`
    );
  }
};

// Function to add an important date
const addImportantDate = (date) => {
  // Format descriptive text for recurrence
  let recurringText = "";
  if (date.isRecurring) {
    if (date.recurrenceType === "custom") {
      recurringText = ` (every ${date.recurrenceInterval} ${date.recurrenceUnit})`;
    } else {
      recurringText = ` (${date.recurrenceType} recurring event)`;
    }
  }

  importantDates.value.push({
    id: Date.now(),
    ...date,
    color: date.isRecurring ? "secondary" : "error", // Different color for recurring events
  });

  saveImportantDates();
  addAuditLog(
    "date",
    `Added ${date.isRecurring ? "recurring" : "important"} date: ${
      date.title
    }${recurringText}`
  );
};

// Function to save important dates to localStorage
const saveImportantDates = () => {
  localStorage.setItem(
    "roll-call-important-dates",
    JSON.stringify(importantDates.value)
  );
};

// Function to add an audit log
const addAuditLog = (action, message) => {
  const log = {
    action,
    message,
    timestamp: Date.now(),
  };
  auditLogs.value.unshift(log);
  localStorage.setItem("roll-call-audit-logs", JSON.stringify(auditLogs.value));
};

// Toggle supervisor mode
const toggleSupervisor = () => {
  isSupervisor.value = !isSupervisor.value;
};

const showList = () => {
  showListView.value = true;
};

const showBarGraphView = () => {
  showListView.value = false;
};

// Modify existing methods to close dialogs after actions
const addUserAndClose = (user) => {
  addUser(user);
  showAddUserDialog.value = false;
};

const addDateAndClose = (date) => {
  addImportantDate(date);
  showImportantDateDialog.value = false;
};

const addPointsAndClose = (userId, points) => {
  addPoints(userId, points);
  showPointsManagerDialog.value = false;
};
</script>
