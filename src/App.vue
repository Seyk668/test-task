<template>
  <TimePicker v-model="selectedTime" format="24h" minTime="08:00" maxTime="22:00" title="Выбрать время" />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import TimePicker from '@/components/TimePicker.vue';

const selectedTime = ref<string>('--:--');
</script>

<style>
body {
  margin: 0;
}
</style>
