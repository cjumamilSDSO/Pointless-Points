<template>
  <div class="calendar">
    <div class="calendar-header">
      <v-btn @click="prevMonth" variant="plain" icon>
        <v-icon> mdi-chevron-left </v-icon>
      </v-btn>
      <h3>{{ months[currentMonth] }} {{ currentYear }}</h3>
      <v-btn @click="nextMonth" variant="plain" icon>
        <v-icon>mdi-chevron-right</v-icon>
      </v-btn>
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
        :class="{
          empty: !dayData.day,
          'current-day': isCurrentDay(dayData.day),
        }"
      >
        <div class="day-number">{{ dayData.day }}</div>
        <div class="day-events">
          <v-chip
            v-for="(event, eventIndex) in dayData.events"
            :key="eventIndex"
            :color="event.color"
            size="x-small"
            class="mt-1"
            :prepend-icon="getEventIcon(event.type)"
            @click="openEventDialog(event, eventIndex, dayData)"
            role="button"
            :class="{ 'cursor-pointer': true }"
          >
            {{ event.title }}
          </v-chip>
        </div>
      </div>
    </div>

    <!-- Event Details Dialog -->
    <EventDetailsDialog
      v-if="selectedEvent"
      :event="selectedEvent"
      :event-id="selectedEventId"
      :current-date="selectedEventDate"
      v-model:showDialog="showEventDialog"
      @update-event="updateEvent"
      @update-single-occurrence="updateSingleOccurrence"
      @delete-event="deleteEvent"
      @delete-single-occurrence="deleteSingleOccurrence"
      @close="closeEventDialog"
    />
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import EventDetailsDialog from "./dialogs/EventDetailsDialog.vue";

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
});

const emit = defineEmits([
  "update-event",
  "delete-event",
  "update-single-occurrence",
  "delete-single-occurrence",
]);

// Dialog control and selected event
const showEventDialog = ref(false);
const selectedEvent = ref(null);
const selectedEventId = ref(null);
const selectedEventDate = ref(null);

const currentDate = new Date();
const currentMonth = ref(currentDate.getMonth());
const currentYear = ref(currentDate.getFullYear());

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
];

const daysInMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value + 1, 0).getDate();
});

const firstDayOfMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value, 1).getDay();
});

const getEventIcon = (type) => {
  switch (type) {
    case "birthday":
      return "mdi-cake-variant";
    case "recurring":
      return "mdi-calendar-sync";
    default:
      return "mdi-star";
  }
};

// Function to check if a date falls on a custom recurrence pattern
const isCustomRecurrenceMatch = (originalDate, currentDate, interval, unit) => {
  // Event must start on or after the original date
  if (currentDate < originalDate) return false;

  // Get integer representations of both dates to ensure correct comparisons
  const origYear = originalDate.getFullYear();
  const origMonth = originalDate.getMonth();
  const origDate = originalDate.getDate();
  const origDay = originalDate.getDay();

  const currYear = currentDate.getFullYear();
  const currMonth = currentDate.getMonth();
  const currDate = currentDate.getDate();
  const currDay = currentDate.getDay();

  switch (unit) {
    case "days":
      // Calculate days between dates
      const dayDiff = Math.round(
        (currentDate - originalDate) / (24 * 60 * 60 * 1000)
      );
      return dayDiff % interval === 0;

    case "weeks":
      // For weekly recurrence: same day of week and the week difference is divisible by interval
      if (origDay !== currDay) return false;

      const weekDiff = Math.floor(
        (currentDate - originalDate) / (7 * 24 * 60 * 60 * 1000)
      );
      return weekDiff % interval === 0;

    case "months":
      // For monthly recurrence: same day of month and month difference is divisible by interval
      if (origDate !== currDate) return false;

      const monthDiff = (currYear - origYear) * 12 + (currMonth - origMonth);
      return monthDiff % interval === 0;

    case "years":
      // For yearly recurrence: same month and day, and year difference is divisible by interval
      if (origMonth !== currMonth || origDate !== currDate) return false;

      const yearDiff = currYear - origYear;
      return yearDiff % interval === 0;

    default:
      return false;
  }
};

const calendarDays = computed(() => {
  const days = [];
  const totalDays = daysInMonth.value;
  const firstDay = firstDayOfMonth.value;

  // Add empty cells for the days before the first day of the month
  for (let i = 0; i < firstDay; i++) {
    days.push({ day: "", events: [] });
  }

  for (let day = 1; day <= totalDays; day++) {
    const events = [];
    const currentDateObj = new Date(currentYear.value, currentMonth.value, day);
    const currentDateKey = currentDateObj.toISOString().split("T")[0]; // YYYY-MM-DD format

    // Add birthday events
    props.users.forEach((user) => {
      if (user.birthdate) {
        const bDate = new Date(user.birthdate);
        if (
          bDate.getMonth() === currentMonth.value &&
          bDate.getDate() === day
        ) {
          events.push({
            type: "birthday",
            title: `${user.name}'s Birthday`,
            color: "primary",
          });
        }
      }
    });

    // Process both one-time and recurring events
    props.importantDates.forEach((date) => {
      const iDate = new Date(date.date);

      // Check for exceptions first
      let hasException = false;
      let exceptionData = null;

      if (date.exceptions && date.exceptions.length > 0) {
        const exception = date.exceptions.find(
          (e) => e.date === currentDateKey
        );
        if (exception) {
          hasException = true;
          exceptionData = exception;
        }
      }

      // Skip this date if it's marked as deleted in exceptions
      if (hasException && exceptionData.type === "deleted") {
        return; // Skip to next date
      }

      // Handle one-time events
      if (
        !date.isRecurring &&
        iDate.getMonth() === currentMonth.value &&
        iDate.getDate() === day &&
        iDate.getFullYear() === currentYear.value
      ) {
        events.push({
          type: "important",
          title: date.title,
          description: date.description,
          color: date.color || "error",
          id: date.id,
        });
      }

      // Handle recurring events
      else if (date.isRecurring) {
        let matches = false;

        if (
          date.recurrenceType === "yearly" &&
          iDate.getMonth() === currentMonth.value &&
          iDate.getDate() === day
        ) {
          matches = true;
        } else if (
          date.recurrenceType === "monthly" &&
          iDate.getDate() === day
        ) {
          matches = true;
        } else if (
          date.recurrenceType === "weekly" &&
          iDate.getDay() === currentDateObj.getDay()
        ) {
          // Only match weekly events if the original date is on or before the current date
          const originalTime = iDate.getTime();
          const currentTime = currentDateObj.getTime();
          if (originalTime <= currentTime) {
            matches = true;
          }
        } else if (
          date.recurrenceType === "custom" &&
          date.recurrenceInterval &&
          date.recurrenceUnit
        ) {
          matches = isCustomRecurrenceMatch(
            iDate,
            currentDateObj,
            date.recurrenceInterval,
            date.recurrenceUnit
          );
        }

        if (matches) {
          // If there's a modified exception for this date, use the exception data
          if (hasException && exceptionData.type === "modified") {
            events.push({
              type: "recurring",
              title: exceptionData.title || date.title,
              description: exceptionData.description || date.description,
              color: exceptionData.color || date.color || "secondary",
              id: date.id,
              isException: true,
              exceptionDate: currentDateKey,
            });
          } else {
            // Otherwise use the standard recurring event data
            events.push({
              type: "recurring",
              title: date.title,
              description: date.description,
              color: date.color || "secondary",
              id: date.id,
              date: date.date,
              isRecurring: true,
              recurrenceType: date.recurrenceType,
              recurrenceInterval: date.recurrenceInterval,
              recurrenceUnit: date.recurrenceUnit,
            });
          }
        }
      }
    });

    days.push({ day, events });
  }

  return days;
});

// Real-time current date for highlighting today
const today = new Date();
const todayDate = today.getDate();
const todayMonth = today.getMonth();
const todayYear = today.getFullYear();

// Function to check if a given day is the current day
const isCurrentDay = (day) => {
  return (
    day === todayDate &&
    currentMonth.value === todayMonth &&
    currentYear.value === todayYear
  );
};

const nextMonth = () => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }
};

const prevMonth = () => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value--;
  } else {
    currentMonth.value--;
  }
};

// Function to open the event details dialog
const openEventDialog = (event, eventIndex, dayData) => {
  // For custom events, we need to find the original event with all properties
  if (event.type !== "birthday") {
    // Find the original event data from importantDates with all the properties needed
    const originalEvent = props.importantDates.find(
      (date) =>
        date.title === event.title && date.description === event.description
    );

    if (originalEvent) {
      selectedEvent.value = originalEvent;
      selectedEventId.value = originalEvent.id;
    } else {
      selectedEvent.value = event;
      selectedEventId.value = event.id;
    }

    // Store the current date of this occurrence (needed for single occurrence operations)
    const day = dayData.day;
    if (day) {
      selectedEventDate.value = new Date(
        currentYear.value,
        currentMonth.value,
        day
      );
    }
  } else {
    // For birthdays, just use the event as is
    selectedEvent.value = event;
    selectedEventId.value = null; // Birthdays can't be edited/deleted
    selectedEventDate.value = null;
  }

  showEventDialog.value = true;
};

// Function to update an event (all occurrences)
const updateEvent = (data) => {
  emit("update-event", data);
};

// Function to update a single occurrence of a recurring event
const updateSingleOccurrence = (data) => {
  emit("update-single-occurrence", data);
};

// Function to delete an event (all occurrences)
const deleteEvent = (id) => {
  emit("delete-event", id);
};

// Function to delete a single occurrence of a recurring event
const deleteSingleOccurrence = (data) => {
  emit("delete-single-occurrence", data);
};

// Function to close the dialog
const closeEventDialog = () => {
  selectedEvent.value = null;
  showEventDialog.value = false;
};
</script>

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
  min-height: 50px;
  padding: 4px;
  border: 1px solid var(--v-border-color);
  border-radius: 4px;
  position: relative;
  overflow: hidden;
}

.calendar-day.current-day {
  background-color: rgba(var(--v-theme-primary), 0.1);
  border: 2px solid rgb(var(--v-theme-primary));
  box-shadow: 0 0 8px rgba(var(--v-theme-primary), 0.4);
}

.calendar-day.current-day .day-number {
  font-weight: bold;
  color: rgb(var(--v-theme-primary));
}

.calendar-day.current-day::after {
  content: "";
  position: absolute;
  top: 4px;
  right: 4px;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: rgb(var(--v-theme-primary));
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
  max-height: calc(100% - 24px);
  overflow-y: auto;
}

.day-events .v-chip {
  margin: 1px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
  font-size: 0.7rem;
  height: auto !important;
  padding: 1px 4px;
}

@media (max-width: 960px) {
  .calendar-day {
    min-height: 60px;
  }

  .day-events .v-chip {
    font-size: 0.65rem;
  }
}

.cursor-pointer {
  cursor: pointer;
}
</style>
