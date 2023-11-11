<!--Создание таблицы, ее редактирование, фильтрация, удаление и добавление данных-->

<script setup>
// Импорты необходимых стилей и модулей.
import '../assets/Grid.css'
import { ref, computed } from 'vue'
import ConfirmDeleteComponent from '../IternalDevices/ConfirmDeleteComponent.vue';

// Определение входных свойств (props) для этого компонента.
const props = defineProps({
  data: Object,
  columns: Array,
  filterKey: String
})

// Устанавливаем начальное значение для ключа сортировки и для порядка сортировки.
const sortKey = ref('')
const sortOrders = ref(
  props.columns.reduce((o, key) => ((o[key] = 1), o), {}) // Создаём объект с ключами из props.columns и значениями 1
)

// Рассчитываем отфильтрованные данные
const filteredData = computed(() => {
  // Деструктуризация объекта props
  let { data, filterKey } = props

  // Если есть ключ для фильтрации, мы фильтруем данные
  if (filterKey) {
    filterKey = filterKey.toLowerCase()
    data = data.filter(row => {
      return Object.keys(row).some(key => {
        return String(row[key]).toLowerCase().indexOf(filterKey) > -1
      })
    })
  }
  
  // Если установлен ключ сортировки, мы сортируем данные
  const key = sortKey.value
  if (key) {
    const order = sortOrders.value[key]
    data = data.slice().sort((a, b) => {
      a = a[key]
      b = b[key]
      return (a === b ? 0 : a > b ? 1 : -1) * order
    })
  }
  return data // Вернуть отфильтрованные/отсортированные данные
})

// Этот метод будет методом сортировки колонки
function sortBy(key) {
  sortKey.value = key
  sortOrders.value[key] *= -1
}

// Простая функция для преобразования первого символа строки в заглавную букву 
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

// Состояния для редактирования
let isEditing = ref(false)
let currentEditingEntryId = ref(null)

// Функция для включения режима редактирования
function editEntry(entry) {
  if (isEditing.value) {
    displayError("Finish editing first!");
  } else {
    entry.editMode = true
    isEditing.value = true
    currentEditingEntryId.value = entry.id
    // После установки режима редактирования, сфокусируемся на элементе input
    setTimeout(() => {
      const inputElement = document.getElementById(`input-${entry.Name}`)
      inputElement.focus()
    }, 0)
  }
}

// Состояние для отображения ошибок
let errorMessage = ref(null)

// Функция для сохранения изменений
function saveEntry(entry) {
  // Запускаем валидацию только если находимся в режиме редактирования
  if (entry.editMode) {
    // Проверяем, содержат ли поля Name и SurName только буквы
    const reg = /^[a-zа-яё]+$/i; 
    if(!reg.test(entry.Name) || !reg.test(entry.SurName)){
      displayError("Name and Surname can only contain letters!");
    }
    // Проверяем, все ли необходимые поля заполнены
    else if (!entry.Name || !entry.SurName || !entry.Standing || !entry.Age || !entry.Address) {
      displayError("All fields are required!");
    }
    // Проверяем, не меньше ли значения "Standing" или "Age" нуля
    else if (entry.Standing < 0 || entry.Age < 1) {
      displayError("Fields 'Standing' and 'Age' can't have values less than 0!");
    }
    // Если ошибок не обнаружено, выходим из режима редактирования
    else {
      entry.editMode = false;
    }

    // Если не найдено никаких ошибок, выходим из режима редактирования
    if (!errorMessage.value) {
      entry.editMode = false;
      isEditing.value = false
      currentEditingEntryId.value = null
    }
  }
}

// Функция для отображения ошибок
function displayError(message) {
  errorMessage.value = message;
  // Удаляем сообщение об ошибке через 2 секунды
  setTimeout(() => {
   errorMessage.value = null;
  }, 2000);
}

// Функция для удаления записи
function deleteEntry(entry) {
  const index = filteredData.value.findIndex(item => item === entry) 
  if (index > -1) {
    if (currentEditingEntryId.value === entry.id) {
      isEditing.value = false
      currentEditingEntryId.value = null
    }
    filteredData.value.splice(index, 1)
  }
}

// Состояние для удаления
let deleteCandidate = ref(null)

// Функция для извлечения кандидата на удаление
function promptDelete(entry) {
  deleteCandidate.value = entry
}

// Функция для подтверждения удаления
function deleteConfirmed() {
  deleteEntry(deleteCandidate.value)
  deleteCandidate.value = null
}

// Функция для отмены удаления
function cancelDelete() {
  deleteCandidate.value = null
}

// Обрабатывает подтверждение удаления
function handleConfirm() {
  if (deleteCandidate.value) {
    deleteEntry(deleteCandidate.value)
    deleteCandidate.value = null
  }
}
</script>

<template>
  <!-- Таблица с данными, если они имеются-->
  <table v-if="filteredData.length">
    <thead>
      <tr>
        <!-- Заголовок для каждой колонки -->
        <th v-for="key in columns"
          @click="sortBy(key)"
          :class="{ active: sortKey === key }">
          <!-- Используется функция capitalize для преобразования первого символа ключа в верхний регистр -->
          {{ capitalize(key) }}
          <!-- Иконка, указывающая направление сортировки -->
          <span class="arrow" :class="sortOrders[key] > 0 ? 'asc' : 'dsc'">
          </span>
        </th>
        <!-- Заголовок для колонки редактирования-->
        <th>Edit</th>
      </tr>
    </thead>
    <tbody>
      <!-- Строка в таблице для каждой записи в данных -->
      <tr v-for="entry in filteredData" :key="entry.id">
        <!-- Для каждого ключа в записи создаём колонку -->
        <td v-for="key in columns">
          <!-- Если мы редактируем эту запись, показываем поля ввода -->
          <template v-if="entry.editMode">
            <input
              v-if="key === 'Standing' || key === 'Age'"
              :id="`input-${entry.id}`"
              v-model="entry[key]"
              @keypress.enter="saveEntry(entry)"
              type="number"
              min="1"
            />
            <input
              v-else
              :id="`input-${entry.id}`"
              v-model="entry[key]"
              @keypress.enter="saveEntry(entry)"
              type="text"
            />
          </template>
          <!-- Если мы не редактируем эту запись, просто отображаем значение -->
          <template v-else>
            {{ entry[key] }}
          </template>
        </td>
        <!-- Столбец для редактирования и удаления записи -->
        <td class="Edit-devices">
          <!-- Если мы в режиме редактирования, отображаем кнопку "Save" и значок удаления-->
          <template v-if="entry.editMode">
            <button class="Button-save" @click="saveEntry(entry)">
              Save
            </button>
            <!-- Иконка удаления -->
            <img class="trash" src="/images/trash.svg" alt="Deleting" @click="promptDelete(entry)">
          </template>
          <!-- Если мы не редактируем, отображаем значок редактирования -->
          <template v-else>
            <img src="/images/edit.svg" alt="edit" @click="editEntry(entry)">
          </template>
        </td>
      </tr>
    </tbody>
  </table>
  <!-- Если данных нет, отображаем сообщение -->
  <p v-else>No matches found...</p>
  <!-- Компонент для подтверждения удаления -->
  <ConfirmDeleteComponent :isActive="deleteCandidate !== null" @confirm="handleConfirm" @cancel="cancelDelete">
  </ConfirmDeleteComponent>
  <!-- Отображаем ошибку, если она присутствует -->
  <div class="notification-wrapper" v-if="errorMessage">
    <div class="notification">
      <span>{{ errorMessage }}</span>
    </div>
  </div>
</template>