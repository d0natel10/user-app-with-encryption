<template>
    <form v-if="props.activeForm === 'updatePassSecure'" class="page__form">
      <div class="page__form-header">
        <h1>Обновите пароль</h1>
      </div>
        <p>Политика безопастности вашего аккаунта изменилась.</p>
        <p>Пожалуйста, обновите пароль</p>
      <div class="page__form-inputs">
        <label>Старый пароль</label>
        <input type="password" v-model="oldPass" placeholder="Старый пароль..."/>
        <label>Придумайте новый пароль</label>
        <input type="password" v-model="setNewPass" placeholder="Ваш новый пароль..."/>
        <label>Повторите пароль</label>
        <input type="password" v-model="repeatPass" placeholder="Повторите пароль..."/>
      </div>
      <div class="page__form-error" :class="{ 'active': 'formError' }">
          <p>{{ errorMess }}</p>
      </div>
      <div class="page__form-formButton">
        <button @click="PostNewPass" type="button">Установить</button>
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
const oldPass = ref('');

const emit = defineEmits();

const validatePass = (password) => {
    const regex = /^(?=.*[a-zA-Z])(?=.*\d)(?=.*[!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~.]).*$/;
    return regex.test(password);
};

const reset = () => {
    setNewPass.value = '';
    repeatPass.value = '';
    errorMess.value = '';
    oldPass.value = '';
}

const close = () => {
    props.closeForm()
    reset();
}

const updatePassword = async (password) => {
    const data = { pass: props.encryptData(password, props.secretPhase) };
    const response = await axios.patch(`https://c428022c5944a6d7.mokky.dev/users/${props.currentAuthUser.id}`, data);
    alert("Пароль успешно установлен!");
    emit('updateData', response.data);
    props.currentAuthUser.pass = password;
    close();
    
};

const PostNewPass = async () => {
    try {
        const isOldPassCorrect = oldPass.value === props.currentAuthUser.pass;

        if (setNewPass.value === repeatPass.value && setNewPass.value !== '' && isOldPassCorrect) {
            if (props.currentAuthUser.isPassOptions && !validatePass(setNewPass.value)) {
                errorMess.value = "Пароль должен содержать буквы, цифры и знаки препинания";
                return;
            }
            await updatePassword(setNewPass.value);
        } else {
            errorMess.value = setNewPass.value === '' ? "Пароль не может быть пустым!" : "Пароли не совпадают!";
            if (!isOldPassCorrect) {
                errorMess.value += " Неверно указан старый пароль.";
            }
        }
    } catch (e) {
        console.log(e.message);
    }
};
</script>