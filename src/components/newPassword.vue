<template>
    <form v-if="props.activeForm === 'changePassword'" class="page__form">
      <div class="page__form-header">
        <h1>Сменить пароль</h1>
        <img @click="close" src="/public/icons/close.svg" alt="close window"/>
      </div>
      <div class="page__form-inputs">
        <label>Новый пароль</label>
        <input type="password" v-model="setNewPass" placeholder="Новый пароль..."/>
        <label>Повторите пароль</label>
        <input type="password" v-model="repeatPass" placeholder="Повторите новый пароль..."/>
      </div>
      <div class="page__form-error" :class="{ 'active': 'formError' }">
          <p>{{ errorMess }}</p>
      </div>
      <div class="page__form-formButton">
        <button @click="PostNewPass" type="button">Сохранить</button>
      </div>
    </form>
</template>

<script setup> 
import { ref, defineProps, defineEmits} from 'vue';
import axios from 'axios';

const props = defineProps({
    activeForm: String,
    currentAuthUser: Object,
    closeForm: Function,
    encryptData: Function,
    secretPhase: String
});

const setNewPass = ref('');
const repeatPass = ref(''); 
const errorMess = ref('');

const emit = defineEmits();

const validatePass = (password) => {
    const regex = /^(?=.*[a-zA-Z])(?=.*\d)(?=.*[!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~.]).*$/;
    return regex.test(password);
};

const reset = () => {
    setNewPass.value = '';
    repeatPass.value = '';
    errorMess.value = '';
}

const close = () => {
    props.closeForm();
    reset();
}

const updatePassword = async (password) => {
    const data = { pass: props.encryptData(password, props.secretPhase) };
    const response = await axios.patch(`https://c428022c5944a6d7.mokky.dev/users/${props.currentAuthUser.id}`, data);
    alert("Пароль успешно изменен!");
    close();
    emit('updateData', response.data);
};

const PostNewPass = async () => {
    try {
        if (setNewPass.value === repeatPass.value && setNewPass.value !== '') {
            if (props.currentAuthUser.isPassOptions && !validatePass(setNewPass.value)) {
                errorMess.value = "Пароль должен содержать буквы, цифры и знаки препинания";
                return;
            }
            await updatePassword(setNewPass.value);
        } else {
            errorMess.value = setNewPass.value === '' ? "Пароль не может быть пустым!" : "Пароли не совпадают!";
        }
    } catch (e) {
        console.log(e.message);
    }
};
</script>