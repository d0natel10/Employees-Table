<!-- Модальное окно для добавления нового сотрудника -->

<script setup>
// Импорт стилей для данного компонента.
import '../assets/modalAdd.css'
import { defineProps, defineEmits} from 'vue';
   
  const props = defineProps({
      show: {
          type: Boolean,
          required: true
      }
  });

  const emits = defineEmits(['update:show', 'add', 'close']);

</script>

<template>
  <!-- Обертка для модального окна с анимацией перехода -->
  <Transition name="modal">
    <div v-if="show" class="modal-mask">
        <div class="modal-container">
            <div class="modal-header">
                <!-- Заголовок модального окна -->
                <slot name="header">
                    <h3>Add New Employee</h3>
                    <!-- Закрытие модального окна при необходимости -->
                    <img @click="$emit('close'); $emit('update:show', false)" src="/images/close.svg" alt="close_window">
                </slot>
            </div>

            <div class="modal-body">
                <!-- Тело модального окна -->
                <slot name="body"></slot>
            </div>
        
            <div class="modal-footer">
                <!-- Нижняя панель модального окна -->
                <slot name="footer">
                    <!-- Кнопка подтверждения для добавления сотрудника-->
                    <button class="modal-default-button" @click="$emit('add'); $emit('update:show', false)">Confirm</button>
                </slot>
            </div>
        </div>
    </div>
  </Transition>
</template>
