<!-- Компонент для отображения таблицы -->

<script setup>
// Импорт стилей для данного компонента.
import './assets/viewOnPage.css'
import { ref } from 'vue'
import ModelsEmployee from './components/ModelsEmployee.vue'

const showModalAdd = ref(false) 

</script>
<template> /*Вывод основной таблицы */
  <div class="container">
    <div class="controls">
      <h1>-Employee Table-</h1>
    </div> <!-- Вывод основной таблицы -->
    <ModelsEmployee 
        :showModalAdd.sync="showModalAdd" 
        @added="showModalAdd = false"
        @closed="showModalAdd = false"
    /><!-- Кнопка для добавления нового сотрудника -->
    <button id="show-modal-add" @click="showModalAdd = true">Add new</button>
  </div>
</template>
