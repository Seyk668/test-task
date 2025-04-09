<script setup lang="ts">
import { ref, onMounted, nextTick, computed } from 'vue';

interface Props {
  modelValue: string;
  format: '12h' | '24h';
  minTime: string;
  maxTime: string;
  title: string;
}

const props = defineProps<Props>();
const emit = defineEmits(['update:modelValue']);

const is12HourFormat = computed(() => props.format === '12h');

const isAM = ref(true);
const selectedHour = ref(0);
const selectedMinute = ref(0);
const selectedAMPM = ref(0);
const isOpenModal = ref(false);

const hourCol = ref<HTMLElement | null>(null);
const minuteCol = ref<HTMLElement | null>(null);
const ampmCol = ref<HTMLElement | null>(null);

const pad = (num: number): string => String(num).padStart(2, '0');

const baseHours = computed(() => {
  if (is12HourFormat.value) {
    return ['12', '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11'];
  }
  return Array.from({ length: 24 }, (_, i) => pad(i));
});

const displayTime = computed(() => {
  const isTimeSelected = !isNaN(selectedHour.value || selectedMinute.value);

  if (!isTimeSelected) {
    return "--:--";
  }

  const hours = Number(selectedHour.value) || 0;
  const minutes = Number(selectedMinute.value) || 0;

  if (is12HourFormat.value) {
    let displayHour = hours;
    if (hours === 0) {
      displayHour = 12;
    } else if (hours > 12) {
      displayHour = hours - 12;
    } else if (hours === 12) {
      displayHour = 12;
    }

    return `${pad(displayHour)}:${pad(minutes)} ${isAM.value ? 'AM' : 'PM'}`;
  }

  return `${pad(hours)}:${pad(minutes)}`;
});


const baseMinutes = computed(() => Array.from({ length: 60 }, (_, i) => pad(i)));
const baseAMPM = ['AM', 'PM'];

const infiniteHours = computed(() => Array(50).fill(baseHours.value).flat());
const infiniteMinutes = computed(() => Array(50).fill(baseMinutes.value).flat());

const openModal = () => {
  isOpenModal.value = !isOpenModal.value;

  if (isOpenModal.value) {
    nextTick(() => {
      const hourIndex = is12HourFormat.value
          ? baseHours.value.indexOf(pad(selectedHour.value))
          : selectedHour.value;

      const minuteIndex = selectedMinute.value;
      const ampmIndex = selectedAMPM.value;

      const scrollTo = (col: HTMLElement | null, index: number, count: number) => {
        if (!col) return;
        const itemHeight = 26;
        const middleIndex = index + count * 10;
        col.scrollTop = middleIndex * itemHeight - col.clientHeight / 6 + itemHeight / 2;
      };

      scrollTo(hourCol.value, hourIndex, baseHours.value.length);
      scrollTo(minuteCol.value, minuteIndex, 60);
      if (ampmCol.value) scrollTo(ampmCol.value, ampmIndex, 2);
    });
  }
};


const onScroll = (type: 'hour' | 'minute' | 'ampm') => {
  const col = type === 'hour' ? hourCol.value :
      type === 'minute' ? minuteCol.value :
          ampmCol.value;

  if (!col) return;

  const itemHeight = 26;
  const centerPosition = col.scrollTop + col.clientHeight / 2;
  const selectedIndex = Math.round(centerPosition / itemHeight) - 2;

  if (type === 'hour') {
    const baseIndex = selectedIndex % baseHours.value.length;
    selectedHour.value = is12HourFormat.value ?
        parseInt(baseHours.value[baseIndex]) :
        baseIndex;
  }
  else if (type === 'minute') {
    selectedMinute.value = selectedIndex % 60;
  }
  else {
    selectedAMPM.value = selectedIndex % 2;
    isAM.value = selectedAMPM.value === 0;
  }

  updateModelValue();
};

const updateModelValue = () => {
  let hour = selectedHour.value;

  if (is12HourFormat.value) {
    if (!isAM.value && hour !== 12) {
      hour += 12;
    } else if (isAM.value && hour === 12) {
      hour = 0;
    }
  }

  const formatted = `${pad(hour)}:${pad(selectedMinute.value)}`;
  emit('update:modelValue', formatted);
};

onMounted(() => {
  if (props.modelValue) {
    const [hStr, mStr] = props.modelValue.includes(' ') ?
        props.modelValue.split(' ')[0].split(':') :
        props.modelValue.split(':');

    let h = parseInt(hStr);
    const m = parseInt(mStr);

    if (is12HourFormat.value) {
      isAM.value = h < 12;
      selectedAMPM.value = isAM.value ? 0 : 1;
      h = h % 12;
      if (h === 0) h = 12;
    }

    selectedHour.value = h;
    selectedMinute.value = m;
  }

  nextTick(() => {
    const hourIndex = is12HourFormat.value ?
        baseHours.value.indexOf(pad(selectedHour.value)) :
        selectedHour.value;

    const minuteIndex = selectedMinute.value;
    const ampmIndex = selectedAMPM.value;

    const scrollTo = (col: HTMLElement | null, index: number, count: number) => {
      if (!col) return;
      const itemHeight = 26;
      const middleIndex = index + count * 10;
      col.scrollTop = middleIndex * itemHeight - col.clientHeight / 2 + itemHeight / 2;
    };

    scrollTo(hourCol.value, hourIndex, baseHours.value.length);
    scrollTo(minuteCol.value, minuteIndex, 60);
    if (ampmCol.value) scrollTo(ampmCol.value, ampmIndex, 2);
  });
});
</script>

<template>
  <div class="picker-header">
    <div class="header-wrapper">
      <span class="title-header" @click="openModal">{{ title }}</span>
      <span>{{ displayTime || '--:--' }}</span>
    </div>
  </div>
  <div v-if="isOpenModal" class="time-picker">
    <div class="overlay" @click.self="openModal"></div>
    <div class="picker-wrapper">
      <div class="picker-overlay"></div>
      <div class="picker-column" ref="hourCol" @scroll="onScroll('hour')">
        <div class="spacer" />
        <div
            v-for="(hour, index) in infiniteHours"
            :key="'h' + index"
            class="picker-item"
        >{{ hour }}</div>
        <div class="spacer" />
      </div>
      <div class="picker-column" ref="minuteCol" @scroll="onScroll('minute')">
        <div class="spacer" />
        <div
            v-for="(minute, index) in infiniteMinutes"
            :key="'m' + index"
            class="picker-item"
        >{{ minute }}</div>
        <div class="spacer" />
      </div>
      <div
          v-if="is12HourFormat"
          class="picker-column ampm-column"
          ref="ampmCol"
          @scroll="onScroll('ampm')"
      >
        <div class="spacer" />
        <div
            v-for="(period, index) in baseAMPM"
            :key="'ampm' + index"
            class="picker-item ampm-item"
        >{{ period }}</div>
        <div class="spacer" />
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.picker-header {
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
}

.header-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding: 8px;
  font-size: 0.8rem;
  background-color: $tg-theme-text-color;
  color: $tg-theme-secondary-bg-color;
}

.title-header {
  cursor: pointer;
}

.time-picker {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  position: absolute;
  top: 0;
  height: 100vh;
  width: 100%;
}

.overlay {
  height: 100%;
  width: 100%;
  position: absolute;
}

.title {
  margin-bottom: 1rem;
  font-size: 1.2rem;
  font-weight: bold;
}

.picker-wrapper {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  position: fixed;
  right: 50%;
  bottom: 0;
  transform: translateX(50%);
  background-color: #111;
  width: 100%;
  padding: 10px 5px;
  height: 200px;
  border-radius: 1.5rem 1.5rem 0 0;
  justify-content: center;
}

.picker-overlay {
  position: absolute;
  top: 50%;
  left: 0;
  width: 100%;
  height: 30px;
  background-color: $tg-theme-hint-color;
  border-radius: 0.4rem;
  margin-top: -14px;
  pointer-events: none;
  z-index: 1;
}

.picker-column {
  height: 120px;
  width: 60px;
  overflow-y: scroll;
  scroll-snap-type: y mandatory;
  scrollbar-width: none;
  -ms-overflow-style: none;
  z-index: 2;
  mask-image: linear-gradient(to bottom, transparent, black 30%, black 70%, transparent);
}

.ampm-column {
  width: 60px;
}

.picker-column::-webkit-scrollbar {
  display: none;
}

.picker-item {
  height: 20px;
  line-height: 20px;
  text-align: center;
  font-size: 1.2rem;
  padding: 3px;
  scroll-snap-align: center;
  color: $tg-theme-secondary-bg-color;
  cursor: pointer;
}

.spacer {
  height: 47px;
}

@media (min-width: 600px) {
  .picker-header {
    padding: 20px;
  }

  .header-wrapper {
    max-width: 400px;
    font-size: 1rem;
    border-radius: 0.4rem;
  }

  .picker-wrapper {
    max-width: 400px;
    padding: 16px;
    border-radius: 0.5rem;
  }

  .picker-column {
    width: 70px;
  }

  .ampm-column {
    width: 80px;
  }

  .picker-item {
    font-size: 1.4rem;
  }
}
</style>
