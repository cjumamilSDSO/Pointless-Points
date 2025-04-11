<template>
  <div class="calendar">
    <div class="calendar-header">
      <v-btn icon="mdi-chevron-left" @click="prevMonth" />
      <h3>{{ months[currentMonth] }} {{ currentYear }}</h3>
      <v-btn icon="mdi-chevron-right" @click="nextMonth" />
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
          'current-day': isCurrentDay(dayData.day)
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
          >
            {{ event.title }}
          </v-chip>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

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
  const originalTime = originalDate.getTime();
  const currentTime = currentDate.getTime();

  // Event must start on or after the original date
  if (currentTime < originalTime) return false;

  // Calculate time difference in milliseconds
  const timeDiff = currentTime - originalTime;

  // Convert interval to milliseconds based on unit
  let unitInMs;
  switch (unit) {
    case "days":
      unitInMs = 24 * 60 * 60 * 1000;
      break;
    case "weeks":
      unitInMs = 7 * 24 * 60 * 60 * 1000;
      break;
    case "months":
      // For months, we need to check if the date matches rather than just ms
      const monthDiff =
        (currentDate.getFullYear() - originalDate.getFullYear()) * 12 +
        (currentDate.getMonth() - originalDate.getMonth());
      return (
        monthDiff % interval === 0 &&
        currentDate.getDate() === originalDate.getDate()
      );
    case "years":
      // For years, check if month and day match, and the year difference is divisible by interval
      const yearDiff = currentDate.getFullYear() - originalDate.getFullYear();
      return (
        yearDiff % interval === 0 &&
        currentDate.getMonth() === originalDate.getMonth() &&
        currentDate.getDate() === originalDate.getDate()
      );
    default:
      return false;
  }

  // For days and weeks, check if the time difference is divisible by the interval
  if (unit === "days" || unit === "weeks") {
    return Math.round(timeDiff / unitInMs) % interval === 0;
  }

  return false;
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
          color: date.color || "error",
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
          events.push({
            type: "recurring",
            title: date.title,
            color: date.color || "secondary",
            description: date.description,
          });
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
  min-height: 80px;
  padding: 4px;
  border: 1px solid var(--v-border-color);
  border-radius: 4px;
  position: relative;
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
  content: '';
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
}

@media (max-width: 960px) {
  .calendar-day {
    min-height: 60px;
  }
}
</style>
