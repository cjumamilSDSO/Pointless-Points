<template>
  <v-dialog v-model="dialog" max-width="500px">
    <v-card>
      <v-card-title class="d-flex pb-1">
        <v-icon :icon="getEventIcon(event.type)" class="mr-2" />
        <div class="text-truncate">{{ event.title }}</div>
        <v-spacer></v-spacer>
        <v-btn icon="mdi-close" size="small" @click="close"></v-btn>
      </v-card-title>

      <v-card-subtitle v-if="event.date">
        {{ formatDate(event.date) }}
        <v-chip
          v-if="event.isRecurring"
          size="x-small"
          color="info"
          class="ml-2"
          variant="outlined"
        >
          {{ getRecurrenceLabel() }}
        </v-chip>
      </v-card-subtitle>

      <v-divider></v-divider>

      <v-card-text>
        <!-- View mode -->
        <div v-if="!isEditing" class="mt-2">
          <p v-if="event.description" class="mb-4">{{ event.description }}</p>
          <p v-else class="text-caption text-grey">No description</p>
        </div>

        <!-- Edit mode -->
        <v-form v-else ref="form" @submit.prevent="saveChanges">
          <v-text-field
            v-model="editedEvent.title"
            label="Event Title"
            required
            dense
          ></v-text-field>

          <v-text-field
            v-model="editedEvent.date"
            label="Date"
            type="date"
            required
            dense
          ></v-text-field>

          <v-textarea
            v-model="editedEvent.description"
            label="Description"
            rows="3"
            auto-grow
            dense
          ></v-textarea>

          <v-switch
            v-if="!isBirthday"
            v-model="editedEvent.isRecurring"
            label="Recurring Event"
            color="primary"
            hide-details
            class="mb-1"
            density="compact"
          ></v-switch>

          <v-expand-transition>
            <div v-if="editedEvent.isRecurring">
              <v-select
                v-model="editedEvent.recurrenceType"
                :items="recurrenceTypes"
                label="Recurrence Type"
                required
                dense
                @update:model-value="handleRecurrenceTypeChange"
              ></v-select>

              <v-expand-transition>
                <div
                  v-if="editedEvent.recurrenceType === 'custom'"
                  class="custom-recurrence"
                >
                  <v-text-field
                    v-model.number="editedEvent.recurrenceInterval"
                    label="Repeat every"
                    type="number"
                    min="1"
                    required
                    dense
                    class="recurrence-interval"
                  />

                  <v-select
                    v-model="editedEvent.recurrenceUnit"
                    :items="recurrenceUnits"
                    label="Unit"
                    required
                    dense
                    class="recurrence-unit"
                  ></v-select>
                </div>
              </v-expand-transition>
            </div>
          </v-expand-transition>
        </v-form>
      </v-card-text>

      <v-card-actions>
        <template v-if="!isEditing && !isBirthday">
          <v-btn
            color="primary"
            prepend-icon="mdi-pencil"
            variant="text"
            @click="startEditing"
          >
            Edit
          </v-btn>
          <v-btn
            color="error"
            prepend-icon="mdi-delete"
            variant="text"
            @click="confirmDelete"
          >
            Delete
          </v-btn>
        </template>
        <template v-else-if="isEditing">
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="text" @click="cancelEdit">
            Cancel
          </v-btn>
          <v-btn color="primary" @click="saveChanges"> Save </v-btn>
        </template>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <!-- Edit Recurring Event Dialog -->
  <v-dialog v-model="editRecurringConfirmation" max-width="450px">
    <v-card>
      <v-card-title class="text-h6">Edit Recurring Event</v-card-title>
      <v-card-text>
        Would you like to edit only this occurrence, or this and all future
        occurrences?
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          color="grey"
          variant="text"
          @click="editRecurringConfirmation = false"
        >
          Cancel
        </v-btn>
        <v-btn
          color="secondary"
          variant="text"
          @click="saveChangesForSingleOccurrence"
        >
          This occurrence only
        </v-btn>
        <v-btn
          color="primary"
          variant="text"
          @click="saveChangesForAllOccurrences"
        >
          All occurrences
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <!-- Delete Confirmation Dialog -->
  <v-dialog v-model="deleteConfirmation" max-width="450px">
    <v-card>
      <v-card-title class="text-h6">Delete Event</v-card-title>
      <v-card-text v-if="event.isRecurring">
        Would you like to delete only this occurrence, or this and all future
        occurrences?
      </v-card-text>
      <v-card-text v-else>
        Are you sure you want to delete this event?
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="grey" variant="text" @click="deleteConfirmation = false">
          Cancel
        </v-btn>
        <template v-if="event.isRecurring">
          <v-btn
            color="secondary"
            variant="text"
            @click="deleteSingleOccurrence"
          >
            This occurrence only
          </v-btn>
          <v-btn color="error" variant="text" @click="deleteAllOccurrences">
            All occurrences
          </v-btn>
        </template>
        <template v-else>
          <v-btn color="error" variant="text" @click="deleteEvent">
            Delete
          </v-btn>
        </template>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { ref, computed, reactive } from "vue";

const props = defineProps({
  event: {
    type: Object,
    required: true,
  },
  eventId: {
    type: [String, Number],
    default: null,
  },
  showDialog: {
    type: Boolean,
    default: false,
  },
  currentDate: {
    type: Date,
    default: null,
  },
});

const emit = defineEmits([
  "update:showDialog",
  "update-event",
  "delete-event",
  "delete-single-occurrence",
  "update-single-occurrence",
  "close",
]);

const dialog = computed({
  get: () => props.showDialog,
  set: (value) => emit("update:showDialog", value),
});

const isEditing = ref(false);
const deleteConfirmation = ref(false);
const editRecurringConfirmation = ref(false);

// Create a copy of the event for editing
const editedEvent = reactive({ ...props.event });

// Options for recurrence settings
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

// Determine if this is a birthday event (which can't be edited or deleted)
const isBirthday = computed(() => props.event.type === "birthday");

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

const formatDate = (dateStr) => {
  const date = new Date(dateStr);
  return date.toLocaleDateString("en-US", {
    weekday: "long",
    year: "numeric",
    month: "long",
    day: "numeric",
  });
};

const getRecurrenceLabel = () => {
  if (!props.event.isRecurring) return "";

  switch (props.event.recurrenceType) {
    case "weekly":
      return "Weekly";
    case "monthly":
      return "Monthly";
    case "yearly":
      return "Yearly";
    case "custom":
      const interval = props.event.recurrenceInterval || 1;
      const unit = props.event.recurrenceUnit || "days";
      return `Every ${interval} ${unit}`;
    default:
      return "Recurring";
  }
};

const startEditing = () => {
  // Create a fresh copy of the event data
  Object.assign(editedEvent, props.event);
  isEditing.value = true;
};

const cancelEdit = () => {
  isEditing.value = false;
};

const saveChanges = () => {
  if (editedEvent.isRecurring) {
    // For recurring events, ask if they want to edit just this occurrence or all
    editRecurringConfirmation.value = true;
  } else {
    // For non-recurring events, just save directly
    emit("update-event", {
      id: props.eventId,
      eventData: { ...editedEvent },
      mode: "all",
    });
    isEditing.value = false;
  }
};

const saveChangesForAllOccurrences = () => {
  emit("update-event", {
    id: props.eventId,
    eventData: { ...editedEvent },
    mode: "all",
  });
  editRecurringConfirmation.value = false;
  isEditing.value = false;
};

const saveChangesForSingleOccurrence = () => {
  emit("update-single-occurrence", {
    id: props.eventId,
    eventData: { ...editedEvent },
    occurrence: props.currentDate,
  });
  editRecurringConfirmation.value = false;
  isEditing.value = false;
};

const confirmDelete = () => {
  deleteConfirmation.value = true;
};

const deleteEvent = () => {
  emit("delete-event", props.eventId);
  deleteConfirmation.value = false;
  dialog.value = false;
};

const deleteSingleOccurrence = () => {
  emit("delete-single-occurrence", {
    id: props.eventId,
    occurrence: props.currentDate,
  });
  deleteConfirmation.value = false;
  dialog.value = false;
};

const deleteAllOccurrences = () => {
  emit("delete-event", props.eventId);
  deleteConfirmation.value = false;
  dialog.value = false;
};

const close = () => {
  dialog.value = false;
  isEditing.value = false;
  emit("close");
};

const handleRecurrenceTypeChange = () => {
  // Reset custom values if switching away from custom
  if (editedEvent.recurrenceType !== "custom") {
    editedEvent.recurrenceInterval = 1;
    editedEvent.recurrenceUnit = "weeks";
  }
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
