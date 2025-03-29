<script setup>
import { ref, h } from 'vue'
import Modal from './Modal.vue'

// Логика открытия/закрытия модального окна
const isOpen = ref(false);

function onClose() {
  isOpen.value = false;
}

// Содержимое модального окна
const orderNumber = 'У00017711'; // Номер заказа
const selectedReasons = ref([]); // Массив для хранения выбранных причин
const reasons = ref([
  'Не подходит дата или время',
  'Не подходит тоннаж груза',
  'Не подходит объем груза',
  'Не подходит тип фургона',
  'Нет пропуска в МКАД ТТК СК',
  'Поломка машины/авария',
  'Заболел',
  'Нет санобработки',
  'Нет мед. книжки',
  'Нет гидроборта',
  'Нет растентовки',
  'Не устраивает ставка за рейс',
  'Не устраивает маршрут',
  'Другое'
]);

// Обработчик изменения состояния чекбокса
function handleCheckboxChange(event, reason) {
  if (event.target.checked) {
    // Если чекбокс отмечен, добавляем причину в массив
    selectedReasons.value.push(reason);
  } else {
    // Если чекбокс снят, удаляем причину из массива
    const index = selectedReasons.value.indexOf(reason);
    if (index > -1) {
      selectedReasons.value.splice(index, 1);
    }
  }
}

// Генерируем заголовок
const title = `Отказаться от выполнения заказа № ${orderNumber}`

// Генерируем чекбоксы
const checkboxesVNode = h('div', { class: 'checkbox-group' }, reasons.value.map(reason =>
  h('label', { class: 'checkbox-label' }, [
    h('input', { 
      type: 'checkbox', 
      class: 'checkbox-input',
      value: reason,
      onChange: (event) => handleCheckboxChange(event, reason) 
    }),
    h('span', { class: 'checkbox-custom' }),
    reason
  ])
));

// Генерируем весь контент
const content = h('div', { class: 'modal__content' }, [
  h('h3', 'Расскажите, почему отказались от выполнения заказа?'),
  checkboxesVNode
]);

// Градиент под курсором
function mouseMoveGradient(event) {
  let elem = event.currentTarget;
  let button = elem.querySelector(".container__button");

  if (button && button.contains(event.target)) {
    elem.style.background = "none";
    return;
  }

  if (elem.getAttribute('class') == "container") {
    let box = elem.getBoundingClientRect();
    let X = event.clientX - box.left;
    let Y = event.clientY - box.top;

    let xPercent = X / (box.width / 100);
    let yPercent = Y / (box.height / 100);
    elem.style.background = `radial-gradient(circle 55px at ${xPercent}% ${yPercent}%, rgba(205, 16, 255, .5), rgba(0, 0, 0, 0))`;
  }
};
function mouseLeave(event) {
  let elem = event.target;
  elem.style.background = "none";
};
</script>

<template>
  <div class="container" 
    @mousemove="mouseMoveGradient" 
    @mouseleave="mouseLeave">
    <button class="container__button" 
      type="button"
      @click="isOpen = true"
    >
      Отказаться от заказа
    </button>
  
    <Modal 
    :isOpen="isOpen"
    @onClose="onClose"
    :title="title"
    :content="content"
    :selectedReasons="selectedReasons"
    />
  </div>
</template>

<style lang="scss" scoped>
.container {
  position: fixed;
  left: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  background: var(--light-color);
  .container__button {
    width: 512.65px;
    height: 68.35px;
  }
}

</style>