# iOS-style TimePicker

Компонент выбора времени с iOS-подобным интерфейсом.

## Скрипты

- `npm run dev` — запуск в режиме разработки
- `npm run build` — сборка
- `npm run test` — запуск тестов

## Использование

```vue
<TimePicker v-model="selectedTime" title="Выбрать время" :format="'24h'" />
```