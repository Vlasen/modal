## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

# Компонент Modal.vue

## Описание
`Modal.vue` — это настраиваемый модальный компонент для Vue 3, который поддерживает:
- Закрытие по клику вне окна
- Анимацию появления (разная для мобильных и десктопных устройств)
- Свайп вниз для закрытия на мобильных устройствах
- Отображение сообщений об ошибках и успешной отправке формы


## Установка и использование

### 1. Импорт компонента

<script setup>
import { ref } from 'vue';
import Modal from '@/components/Modal.vue';

const isModalOpen = ref(false);
const selectedReasons = ref([]);

function onClose() {
  isOpen.value = false;
}
</script>

<template>
  <Modal 
    :isOpen="isModalOpen"
    :title="'Заголовок модального окна'"
    :content="'Контент модального окна'"
    :selectedReasons="selectedReasons"
    @onClose="onClose"
  />
</template>