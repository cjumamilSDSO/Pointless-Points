<template>
  <div class="bar-graph-container">
    <div class="bars-container">
      <div v-for="user in sortedUsers" :key="user.id" class="bar-item">
        <div class="bar-wrapper">
          <div
            class="bar"
            :style="{
              height: `${(user.points / maxPoints) * 300}px`,
              backgroundColor: user.color || '#90CAF9',
            }"
          ></div>
        </div>
        <div class="bar-labels">
          <div class="user-name">{{ user.name }}</div>
          <div class="points">{{ user.points }} points</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  users: {
    type: Array,
    required: true,
  },
});

const sortedUsers = computed(() => {
  return [...props.users].sort((a, b) => b.points - a.points);
});

const maxPoints = computed(() => {
  return Math.max(...props.users.map((u) => u.points), 1);
});
</script>

<style scoped>
.bar-graph-container {
  width: 100%;
  padding: 2rem;
  min-height: 400px;
}

.bars-container {
  display: flex;
  justify-content: center;
  gap: 2rem;
  height: 100%;
}

.bar-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 20%;
  max-width: 120px;
}

.bar-wrapper {
  display: flex;
  align-items: flex-end;
  height: 300px; /* Fixed height for bar container */
  width: 100%;
  margin-bottom: 1rem;
}

.bar-labels {
  text-align: center;
}

.user-name {
  font-weight: bold;
  margin-bottom: 0.25rem;
}

.points {
  color: rgba(0, 0, 0, 0.6);
  font-size: 0.9rem;
}

.bar {
  width: 100%;
  min-height: 40px;
  border-radius: 8px 8px 0 0;
  transition: height 0.3s ease;
}
</style>
