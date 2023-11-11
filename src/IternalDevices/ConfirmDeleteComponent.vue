<!-- Окно для подтверждения удаления сотрудника -->

<script>
// Импорт стилей для данного компонента.
import '../assets/delComponent.css'

import { ref, toRefs } from 'vue'

export default {
  // Определение свойства isActive, которое будет передаваться в этот компонент. 
  // Это свойство определяет, активно ли модальное окно или нет.
  props: {
    isActive: {
      type: Boolean,
      required: true
    },
  },
  setup(props, { emit }) {
    // Преобразование реактивного свойства isActive в референс.
    const { isActive } = toRefs(props) 

    // Функция, вызываемая при подтверждении удаления.
    const confirmDeletion = () => {
      emit('confirm')
    }

    // Функция, вызываемая при отмене удаления. 
    const cancelDeletion = () => {
      emit('cancel')
    }

    // функции обратного вызова экспортируются, чтобы они могли быть использованы в шаблоне.
    return {
      confirmDeletion,
      cancelDeletion
    }
  }
}
</script>

<template>
  <!-- В случае если isActive является истиной, модальное окно активируется/видимо -->
  <div class="modal-mask" v-if="isActive">
    <div class="modal">
      <p>Are you sure you want to delete this entry?</p>
      <!-- Область кнопок для подтверждения или отмены операции. -->
      <div class ="buttons">
        <button class="modal-button" @click="confirmDeletion">Delete</button>
        <button class="modal-button" @click="cancelDeletion">Cancel</button>
      </div>
    </div>
  </div>
</template> 