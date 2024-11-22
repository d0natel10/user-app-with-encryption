<template>
    <form v-if="props.activeForm === 'addUser'" class="page__form">
      <div class="page__form-header">
        <h1>Добавить пользователя</h1>
        <img @click="close" src="/public/icons/close.svg" alt="close window"/>
      </div>
      <div class="page__form-inputs">
        <label>Имя пользователя</label>
        <input v-model="userName" placeholder="Имя пользвателя..."/>
        <div class="page__form-checkbox">
          <label class="page__form-checkbox-label" for="PassCheck">Парольное ограничение</label>
          <input v-model="passOptions" class="page__form-checkbox-input" type="checkbox" id="PassCheck" name="myCheckbox">
        </div>
      </div>
      <div class="page__form-error" :class="{ 'active': 'formError' }">
          <p>{{ errorMess }}</p>
      </div>
      <div class="page__form-formButton">
        <button @click="AddNewUser" type="button">Добавить</button>
      </div>
    </form>
</template>

<script setup>
    import {ref, defineProps, defineEmits} from 'vue';
    import axios from 'axios';

    const props = defineProps({
        activeForm: String,
        closeForm: Function,
        data: Object,
        encryptData: Function,
        secretPhase: String
    })

    const passOptions = ref(false);
    const userName = ref('');
    const errorMess = ref('')

    const close = () => {
        errorMess.value = '';
        props.closeForm();
        userName.value = '';
    }

    const checkUsers = (username) => {
        const user = props.data.find(use => username === use.name);
        if (!user) {
            return true;
        }
        else {
            return false;
        }
    }

    const emit = defineEmits();
    const AddNewUser = async () => {
        if (userName.value === ''){
            errorMess.value = 'Поле не может быть пустым';
        }
        else if (!checkUsers(userName.value))
        { 
            errorMess.value = 'Такой пользователь уже существует';
        } 
        else {
            try {
                errorMess.value = '';
            const Newdata = {
                name: props.encryptData(userName.value, props.secretPhase),
                pass: props.encryptData('', props.secretPhase),
                isBlocked: props.encryptData(false, props.secretPhase),
                isPassOptions: props.encryptData(passOptions.value, props.secretPhase)
            }
            const response = await axios.post('https://c428022c5944a6d7.mokky.dev/users', Newdata);
            alert('Пользователь успешно создан!');
            close();
            emit('updateDataBase', response.data);
            
        }
        catch (error) {
            console.log(error.message);
        }
    }
}
</script>