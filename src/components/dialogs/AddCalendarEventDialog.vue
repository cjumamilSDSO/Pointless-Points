<template>
  <v-form @submit.prevent="addDate">
    <v-text-field v-model="newDate.title" label="Event Title" required dense />

    <v-text-field v-model="newDate.date" label="Date" type="date" required dense />

    <v-textarea
      v-model="newDate.description"
      label="Description"
      rows="2"
      dense
      auto-grow
    ></v-textarea>

    <v-switch
      v-model="newDate.isRecurring"
      label="Recurring Event"
      color="primary"
      hide-details
      class="mb-1"
      density="compact"
    ></v-switch>

    <v-expand-transition>
      <div v-if="newDate.isRecurring">
        <v-select
          v-model="newDate.recurrenceType"
          :items="recurrenceTypes"
          label="Recurrence Type"
          required
          @update:model-value="handleRecurrenceTypeChange"
          dense
        ></v-select>

        <v-expand-transition>
          <div v-if="newDate.recurrenceType === 'custom'" class="custom-recurrence">
            <v-text-field
              v-model.number="newDate.recurrenceInterval"
              label="Repeat every"
              type="number"
              min="1"
              required
              class="recurrence-interval"
              dense
            />

            <v-select
              v-model="newDate.recurrenceUnit"
              :items="recurrenceUnits"
              label="Unit"
              required
              class="recurrence-unit"
              dense
            ></v-select>
          </div>
        </v-expand-transition>
      </div>
    </v-expand-transition>

    <v-btn color="primary" type="submit" block class="mt-2" size="small"> Add Event </v-btn>
  </v-form>
</template>

<script setup>
import { ref } from "vue";

const emit = defineEmits(["add-date"]);

const recurrenceTypes = [
  { title: "Weekly", value: "weekly" },
  { title: "Monthly", value: "monthly" },
  { title: "Yearly", value: "yearly" },
  { title: "Custom", value: "custom" },
];

const recurrenceUnits = [
  { title: "Day(s)", value: "days" },
  { title: "Week(s)", value: "weeks" },
  { title: "Month(s)", value: "months" },
  { title: "Year(s)", value: "years" },
];

const newDate = ref({
  title: "",
  date: "",
  description: "",
  isRecurring: false,
  recurrenceType: "yearly",
  recurrenceInterval: 1,
  recurrenceUnit: "weeks", // Changed default to weeks
});

const handleRecurrenceTypeChange = () => {
  // Reset custom values if switching away from custom
  if (newDate.value.recurrenceType !== "custom") {
    newDate.value.recurrenceInterval = 1;
    newDate.value.recurrenceUnit = "weeks"; // Changed default to weeks
  }
};

const addDate = () => {
  if (!newDate.value.title || !newDate.value.date) return;

  emit("add-date", {
    title: newDate.value.title,
    date: newDate.value.date,
    description: newDate.value.description,
    isRecurring: newDate.value.isRecurring,
    recurrenceType: newDate.value.isRecurring
      ? newDate.value.recurrenceType
      : null,
    recurrenceInterval:
      newDate.value.isRecurring && newDate.value.recurrenceType === "custom"
        ? newDate.value.recurrenceInterval
        : null,
    recurrenceUnit:
      newDate.value.isRecurring && newDate.value.recurrenceType === "custom"
        ? newDate.value.recurrenceUnit
        : null,
  });

  // Reset form
  newDate.value = {
    title: "",
    date: "",
    description: "",
    isRecurring: false,
    recurrenceType: "yearly",
    recurrenceInterval: 1,
    recurrenceUnit: "weeks", // Changed default to weeks
  };
};
</script>

<style scoped>
.custom-recurrence {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  margin-bottom: 8px;
}

.recurrence-interval {
  flex: 0 0 100px;
}

.recurrence-unit {
  flex: 1;
}
</style>
