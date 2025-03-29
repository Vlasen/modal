<script setup>
import { ref, onMounted, computed } from 'vue'

const props = defineProps({
  isOpen: Boolean,
  title: [String, Object],
  content:  [String, Object],
  selectedReasons: Object,
});
const emit = defineEmits(['onClose']);
const isMobile = ref(false);

// Проверка устройства
const checkIfMobile = () => {
  isMobile.value =
    /Android|iPhone|iPad|iPod|Windows Phone/i.test(navigator.userAgent) || // Проверка User-Agent
    navigator.maxTouchPoints > 1 || // Проверка мультитача
    window.matchMedia('(pointer: coarse)').matches; // Проверка тачскрина
};

// Обработчик изменения размера экрана
const onResize = () => {
  checkIfMobile();
};

onMounted(() => {
  checkIfMobile();
  window.addEventListener('resize', onResize);
});

// Логика свайпа вниз для закрытия модального окна
const startY = ref(0); // Координата начала свайпа
const currentY = ref(0); // Текущая координата пальца во время свайпа
const isSwiping = ref(false); // Флаг свайпа

// Вызывается при первом касании экрана
const startSwipe = (event) => { 
  startY.value = event.touches[0].clientY; // Сохраняем начальное положение пальца по оси Y
  currentY.value = startY.value; // Устанавливаем текущую координату в координату начала свайпа
  isSwiping.value = true;
};

// Вызывается при каждом движении пальца по экрану
const moveSwipe = (event) => { 
  if (!isSwiping.value) return;
  currentY.value = event.touches[0].clientY;
  const deltaY = currentY.value - startY.value;

  if (deltaY > 0) { // Если разница между текущей и начальной координатой положительная, то модальное окно опускается
    document.querySelector('.modal').style.transform = `translateY(${deltaY}px)`;
  }
};

// Вызывается при окончании свайпа
const endSwipe = () => {
  const deltaY = currentY.value - startY.value;
  const modal = document.querySelector('.modal');

  if (deltaY > 150) { // Если свайп был достаточно длинным (более 100px), то закрываем окно
    modal.style.transform = `translateY(100vh)`;
    modal.addEventListener('transitionend', () => {
      emit('onClose');
    });
  } else {
    // Если свайп был недостаточно длинным, возвращаем обратно
    modal.style.transform = 'translateY(0)';
  }

  isSwiping.value = false;
};

// Логика вывода сообщения о статусе отправки данных
const successSatus = ref('');
const errorSatus = ref('');

function submitForm () {
  const element = document.querySelector('.status-box');
  const button = document.querySelector('.submit-button');
  button.setAttribute('disabled', 'true'); // Делаем кнопку неактивной

  if (!isReasons()) { // Если причины не выбраны
    setTimeout(() => {
      element.classList.remove('show'); // Скрываем сообщение
      errorSatus.value = '';
      button.removeAttribute('disabled'); // Делаем кнопку активной
    }, 2000);
  } else { // Если причины выбраны
    setTimeout(() => {
      element.classList.remove('show');
      successSatus.value = '';
      emit('onClose');
      // isCooldown.value = false; 
      button.removeAttribute('disabled');
    }, 2000);
  }
};

// Проверка выбранных пользователем причин
const isReasons = () => {
  const element = document.querySelector('.status-box');
  if (props.selectedReasons.length < 1) { //Если ни одна причина не выбрана, выводим ошибку
    errorSatus.value = 'Выберите причину отказа';
    element.classList.add('show');
    return false
  } else { // Если хотя бы одна причина выбрана, закрываем окно и очищаем массив выбранных причин
    successSatus.value = 'Данные отправлены!';
    element.classList.add('show');
    props.selectedReasons.length = 0;
    return true
  }
};
</script>

<template>
  <Teleport to="body">
    <!-- Темный фон под модальным окном -->
    <div v-if="isOpen"
      class="modal-overlay" ref="overlay"
      @click.self="$emit('onClose')" 
    >
      <Transition :name="isMobile ? 'modal-slide' : 'modal-fade'" appear>
        <!-- Модальное окно -->
        <div class="modal" ref="modal" 
          @touchstart="startSwipe" 
          @touchmove="moveSwipe" 
          @touchend="endSwipe">

          <!-- Кнопка закрытия окна -->
          <div class="modal__close-btn-container">
            <div class="close-btn" @click="$emit('onClose')">
              <svg width="41.77" height="41.77" viewBox="0 0 43 43" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M29.4573 16.4852C30.2244 15.7703 30.2667 14.5688 29.5517 13.8017C28.8367 13.0346 27.6353 12.9924 26.8682 13.7073L21.3123 18.8855L16.1341 13.3297C15.4192 12.5626 14.2177 12.5203 13.4506 13.2352C12.6835 13.9502 12.6413 15.1517 13.3562 15.9188L18.5344 21.4746L12.9786 26.6528C12.2115 27.3678 12.1692 28.5692 12.8841 29.3363C13.5991 30.1034 14.8006 30.1457 15.5677 29.4307L21.1235 24.2525L26.3017 29.8084C27.0167 30.5755 28.2181 30.6178 28.9852 29.9028C29.7523 29.1878 29.7946 27.9864 29.0796 27.2193L23.9014 21.6634L29.4573 16.4852Z" fill="#58595B"/>
                <path fill-rule="evenodd" clip-rule="evenodd" d="M0.332275 21.5691C0.332275 10.0342 9.68314 0.683372 21.218 0.683372C32.7529 0.683372 42.1038 10.0342 42.1038 21.5691C42.1038 33.104 32.7529 42.4549 21.218 42.4549C9.68314 42.4549 0.332275 33.104 0.332275 21.5691ZM21.218 38.6575C11.7804 38.6575 4.12968 31.0068 4.12968 21.5691C4.12968 12.1315 11.7804 4.48078 21.218 4.48078C30.6557 4.48078 38.3064 12.1315 38.3064 21.5691C38.3064 31.0068 30.6557 38.6575 21.218 38.6575Z" fill="#58595B"/>
              </svg>
            </div>
          </div>

          <!-- Заголовок -->
          <component v-if="typeof title === 'object'" :is="title"/>
          <h1 v-else>{{ title }}</h1>
          
          <!-- Контент -->
          <component v-if="typeof content === 'object'" :is="content"/>
          <label v-else>{{ content }}</label>

          <!-- Футер -->
          <footer class="modal__footer">
            <button class="submit-button" 
              type="submit"
              @click="submitForm"
            >
              Отправить
            </button>
          </footer>
        </div> 
      </Transition>
    </div>

    <!-- Вывод статуса отправки данных -->
    <div class="status-box">
      <div v-show="successSatus"
        class="success-conteiner"
      >{{ successSatus }}</div>

      <div v-show="errorSatus"
        class="error-conteiner"
      >{{ errorSatus }}</div>
      
    </div>
  </Teleport>
</template>

<style lang="scss" scoped>
.modal-overlay {
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, .4);
  z-index: 10;
  .modal {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: min(614px, 100%);
    min-width: 320px;
    height: 100%;
    max-height: 100vh;
    margin: 0 auto;
    background: var(--light-color);
    border-radius: 45.57px 45.57px 0 0;
    transition: transform 0.3s ease-out, opacity 0.3s ease-out;
    box-shadow: 0 0 25px rgba(0, 0, 0, .3);
    box-sizing: border-box;

    .modal__close-btn-container {
      display: flex;
      justify-content: flex-end;
      width: calc(100% - 66px);
      height: 42px;
      margin-top: 22.78px;

      .close-btn {
        height: 42px;
        width: 42px;
        cursor: pointer;
      }
      .close-btn:hover {
        opacity: .7;
      }
      .close-btn:active {
        opacity: .9;
      }
    }
    .modal__footer {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 145px;
      background: var(--light-color);

      .submit-button {
        width: 512.65px;
        height: 68.35px;
        @media screen and (max-width: 475px) {
          height: 60px;
        }
        @media screen and (max-width: 353px) {
          height: 50px;
        }
      }
      .submit-button:disabled {
        background: var(--dark-white);  
        color: var(--button-text-color);  
      }
    }
  } 

  /* Анимация появления модального окна для десктопов */
  .modal-fade-enter-active, .modal-fade-leave-active {
    transition: opacity 0.3s ease-out;
  }
  .modal-fade-enter-from, .modal-fade-leave-to {
    opacity: 0;
  }
  
  /* Анимация всплытия снизу для мобильных */
  .modal-slide-enter-active, 
  .modal-slide-leave-active {
    transition: transform 0.3s ease-out, opacity 0.3s ease-out;
  
  }
  .modal-slide-enter-from,
  .modal-slide-leave-to {
    transform: translateY(100%);
    opacity: 0;
  }
  .modal-slide-leave-from,
  .modal-slide-enter-to {
    transform: translateY(0);
    opacity: 1;
  }
}

// Стили статуса отправки данных
.status-box {
  position: fixed;
  left: calc(50vw - 400px / 2);
  bottom: 90px;
  width: auto;
  height: auto;
  text-align: center;
  font-weight: 400;
  font-size: 25.6325px;
  line-height: 28px;
  z-index: 50;
  transition: transform 0.5s ease, opacity 0.5s ease;
  @media screen and (max-width: 768px) {
    left: calc(50vw - 350px / 2);
    font-size: 20px;
    line-height: 24px;
  }
  @media screen and (max-width: 360px) {
    left: calc(50vw - 250px / 2);
    font-size: 18px;
    line-height: 18px;
    bottom: 60px;
  }
  .success-conteiner {
    width: 350px;
    height: 100%;
    padding: 20px 30px;
    background: var(--success-color);
    border-radius: 15px;
    @media screen and (max-width: 768px) {
      width: 300px;
    }
    @media screen and (max-width: 360px) {
      width: 200px;
    }
  }
  .error-conteiner {
    @extend .success-conteiner;
    background: var(--error-color) !important;
  }
}
// Анимация всплытия сообщения о статусе отправки данных
.status-box.show {
  transform: translateY(-40px);
  opacity: 1;
}
</style>