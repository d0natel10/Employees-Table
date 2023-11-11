<!--Начальные данные о таблице, проверка ошибок на ввод-->

<script setup>
// Импорт необходимых модулей и компонентов
import '../assets/models.css'
import ModalAdd from '../IternalDevices/ModalAdd.vue';
import GridOfEmployee from '../ForTable/GridOfEmployee.vue';
import { ref, defineProps, defineEmits } from 'vue'

// Определение свойств компонента
const props = defineProps({
    showModalAdd: {
        type: Boolean,
        default: false
    }
})

// Определение пользовательских событий в компоненте
const emits = defineEmits(['added', 'closed'])

// Определение колонок таблицы и начальных данных
const gridNameColumns = ['Name', 'SurName', 'Standing', 'Age', 'Address']
const searchEmployee = ref('')
const ListOfEmployee = ref([
    // Начальные данные
    { Name: 'Alex', SurName: 'Albon', Standing: 3, Age: 22, Address: 'Bukova street, 32' },
    { Name: 'John', SurName: 'Snow', Standing: 5, Age: 25, Address: 'WinterFall, 4' },
    { Name: 'Dafna', SurName: 'Fairy', Standing: 1.5, Age: 18, Address: 'Forest street, 1' },
    { Name: 'Lando', SurName: 'Norris', Standing: 2, Age: 19, Address: 'Great Britan street, 77' }
])

// Создание объекта нового сотрудника и начальной валидации
const newEmployee = ref({
    Name: '',
    SurName: '',
    Standing: null,
    Age: null,
    Address: ''
})

// Ошибки валидации для каждого поля формы
const nameError = ref('')
const surNameError = ref('')
const standingError = ref('')
const ageError = ref('')
const addressError = ref('')

// Валидация формы добавления нового сотрудника
const validateEmployee = () => {
    let textRegex = /^[a-zа-яё]+$/i
    let numberRegex = /^[1-9]\d*$/
    let standingRegex = /^[0-9]\d*$/

    // Проверка каждого поля
    if (!textRegex.test(newEmployee.value.Name)) {
        nameError.value = 'Only letters are allowed!'
    } else {
        nameError.value = ''
    }

    if (!textRegex.test(newEmployee.value.SurName)) {
        surNameError.value = 'Only letters are allowed!'
    } else {
        surNameError.value = ''
    }

    if (!standingRegex.test(newEmployee.value.Standing)) {
        standingError.value = 'Only positive numbers are allowed!'
    }  else {
        standingError.value = ''
    }

    if (!numberRegex.test(newEmployee.value.Age)) {
        ageError.value = 'Only positive numbers are allowed!'
    } else {
        ageError.value = ''
    }

    if (!newEmployee.value.Address) {
        addressError.value = 'Address field is required!'
    } else {
        addressError.value = ''
    }
    
     // Если нет ошибок валидации, то возвращаем true
    if(!nameError.value && !surNameError.value &&
        !standingError.value && !ageError.value && !addressError.value) {
        return true;
    } else {
        return false;
    }
}

// Добавление нового сотрудника в список после валидации формы
const addEmployee = () => {
    if(validateEmployee()) {
        ListOfEmployee.value.push({ ...newEmployee.value });
        // Сброс полей формы после добавления сотрудника
        newEmployee.value = { Name: '', SurName: '', Standing: null, Age: null, Address: '' };
        props.showModalAdd = false;
        emits('added');
    }
}

// Закрытие модального окна
const closeWindow = () =>{
    props.showModalAdd = false;
    emits('closed')
}
</script>

<template>
    <!--Форма поиска информации о сотрудниках--> 
    <form id="search" class="SearchForm">
    <div class="search-bar">
        <img class="search-icon" src="/images/search.svg" alt="lupa">
        <input class="Search" placeholder="Input some info..." name="query" v-model="searchEmployee">
    </div>
    </form>

    <!--Грид с информацией о сотрудниках-->
    <GridOfEmployee
        :data="ListOfEmployee"
        :columns="gridNameColumns"
        :filter-key="searchEmployee"
    ></GridOfEmployee>

    <!--Модальное окно добавления нового сотрудника--> 
    <ModalAdd :show.sync="showModalAdd" @add="addEmployee" @close="closeWindow">
        <template #body>
        <!--Форма добавления нового сотрудника--> 
        <div class="input-container">
            <div class="input-group">
                <label>
                    Name:
                    <input type="text" placeholder="Input name..." v-model="newEmployee.Name">
                </label>
                <p class="error-message" v-if="nameError">{{ nameError }}</p>
            </div>
        <div class="input-group">
            <label>
                Surname:
                <input type="text" placeholder="Input surname..." v-model="newEmployee.SurName">
            </label>
            <p class="error-message" v-if="surNameError">{{ surNameError }}</p>
        </div>
        <div class="input-group">
            <label>
                Standing:
                <input type="text" placeholder="Input standing..." v-model="newEmployee.Standing">
            </label>
            <p class="error-message" v-if="standingError">{{ standingError }}</p>
        </div>
        <div class="input-group">
            <label>
                Age:
                <input type="text" placeholder="Input age..." v-model="newEmployee.Age">
            </label>
            <p class="error-message" v-if="ageError">{{ ageError }}</p>
        </div>
        <div class="input-group">
            <label>
                Address:
                <input type="text" placeholder="Input address..." v-model="newEmployee.Address">
            </label>
            <p class="error-message" v-if="addressError">{{ addressError }}</p>
        </div> 
    </div>
        </template>
    </ModalAdd>

</template>