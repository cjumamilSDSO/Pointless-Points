<script setup>
defineProps({
  logs: {
    type: Array,
    required: true
  }
});
</script>

<template>
  <div class="audit-log">
    <v-list v-if="logs.length > 0">
      <v-list-item
        v-for="(log, index) in logs"
        :key="index"
        :subtitle="new Date(log.timestamp).toLocaleString()"
        :prepend-icon="getActionIcon(log.action)"
        :color="getActionColor(log.action)"
      >
        {{ log.message }}
      </v-list-item>
    </v-list>
    <v-alert
      v-else
      type="info"
      text="No activity recorded yet."
      class="mb-4"
    ></v-alert>
  </div>
</template>

<script>
export default {
  methods: {
    getActionIcon(action) {
      switch (action) {
        case 'add': return 'mdi-account-plus';
        case 'remove': return 'mdi-account-remove';
        case 'points': return 'mdi-star';
        default: return 'mdi-information';
      }
    },
    getActionColor(action) {
      switch (action) {
        case 'add': return 'success';
        case 'remove': return 'error';
        case 'points': return 'primary';
        default: return 'grey';
      }
    }
  }
}
</script>

<style scoped>
.audit-log {
  max-height: 400px;
  overflow-y: auto;
}
</style>