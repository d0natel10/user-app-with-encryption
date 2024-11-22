<template>
  <section class="window">
  <header class="header">
    <div class="header__profile">
      <img src="/public/icons/icon-h.svg" alt="acc pic" />
      <h1>{{ currentAuthUser ? currentAuthUser.name : '' }}</h1>
    </div>
    <nav class="header__menuSelects">
      <select @change="handleSelects" name="settings">
        <option disabled selected value="settings">Настройки</option>
        <option v-if="currentAuthUser?.pass === ''" value="setPass">Установить пароль</option>
        <option :disabled="!isAuth || currentAuthUser?.pass === '' || activeForm === 'updatePassSecure' || activeForm === 'blockUser'" value="changePassword">Сменить пароль</option>
        <option v-if="currentAuthUser?.name === 'Admin'" :disabled="!isAuth || currentAuthUser?.pass === '' || activeForm === 'updatePassSecure' || activeForm === 'blockUser'" value="addUser">Добавить пользователя</option>
        <option v-if="currentAuthUser?.name === 'Admin'" :disabled="!isAuth || currentAuthUser?.pass === '' || activeForm === 'updatePassSecure' || activeForm === 'blockUser'" value="allUsers">Все пользователи</option>
        <option :hidden="!isAuth" value="leaveAcc">Выход</option>
      </select>
      <button :disabled="activeForm === 'updatePassSecure' || activeForm === 'blockUser' || activeForm === 'passCrypto'" @click="handleAbout" class="header__button">о программе</button>
    </nav>
  </header>
    <section class="page">
      <button v-if="!currentAuthUser && !activeForm" @click="OpenAuthForm" class="page__signButton">
      Вход в систему
      </button>
     
    <form v-if="activeForm === 'auth'" class="page__form">
      <div class="page__form-header">
        <h1>Вход в систему</h1>
        <img @click="closeForm" src="/public/icons/close.svg" alt="close window"/>
      </div>
      <div class="page__form-inputs">
        <label>Имя пользователя</label>
        <input v-model="currentName" placeholder="Имя пользователя..."/>
        <label>Пароль</label>
        <input v-model="currentPass" type="password" placeholder="Пароль..."/>
      </div>
        <div class="page__form-error">
          <p>{{ errorMes }}</p>
          <h4>Попыток осталось: {{ passCounter }}</h4>
        </div>
      <div class="page__form-formButton">
        <button type="button" @click="Authorize">Войти</button>
      </div>
    </form>

    <form v-if="activeForm === 'passCrypto'" class="page__form">
      <div class="page__form-header">
        <h1>Введите парольную фразу для расшифровки данных</h1>
      </div>
      <div class="page__form-inputs">
        <label>Парольная фраза</label>
        <input v-model="currentPhrase" placeholder="Ваша фраза..."/>
      </div>
        <div class="page__form-error">
          <p>{{ errorMes }}</p>
        </div>
      <div class="page__form-formButton">
        <button type="button" @click="CheckCodeOfValide">Подтвердить</button>
      </div>
    </form>

    <NewPassword :encryptData="encryptData" :secretPhase="secretPhase" @updateData="handleCheckUpdate" :activeForm="activeForm" :currentAuthUser="currentAuthUser" :closeForm="closeForm"/>

    <addUser :encryptData="encryptData" :secretPhase="secretPhase" @updateDataBase="handleCheckUpdate" :data="data" :activeForm="activeForm" :closeForm="closeForm"/>

    <allUsers :encryptData="encryptData" :secretPhase="secretPhase" @updateData="handleCheckUpdate" :activeForm="activeForm" :data="data" :closeForm="closeForm"/>

    <setPassword :encryptData="encryptData" :secretPhase="secretPhase" @updateData="handleCheckUpdate" :activeForm="activeForm" :currentAuthUser="currentAuthUser" :closeForm="closeForm"/>

    <updatePass :encryptData="encryptData" :secretPhase="secretPhase" @updateData="handleCheckUpdate" :activeForm="activeForm" :currentAuthUser="currentAuthUser" :closeForm="closeForm"/>

    <blockWindow :activeForm="activeForm"/>

    <aboutProgram :activeForm="activeForm" :closeForm="closeForm"/>
    </section>
  </section>
</template>

<script setup>
  import {watch, onMounted, ref} from 'vue';
  import axios from 'axios';
  import CryptoJS from 'crypto-js';
  import NewPassword from './components/newPassword.vue';
  import addUser from './components/addUser.vue';
  import allUsers from './components/allUsers.vue';
  import aboutProgram from './components/aboutProgram.vue';
  import setPassword from './components/setPassword.vue';
  import updatePass from './components/updatePass.vue';
  import blockWindow from './components/blockWindow.vue';

  const activeForm = ref('passCrypto');
  const currentAuthUser = ref(null);
  const isAuth = ref(false);
  const currentName = ref('');
  const currentPass = ref('');
  const data = ref([]);
  const errorMes = ref('');
  const passCounter = ref(3);
  const currentPhrase = ref('');
  const isCodeSuccess = ref(false);
  const secretPhase = 'qeer';

  const closeForm = () => {
    activeForm.value = null;
    resetOptions();
  }

  const resetOptions = () => {
    currentName.value = '';
    currentPass.value = '';
    errorMes.value = '';
  }

  const OpenAuthForm = () => {
    activeForm.value = 'auth';
  }

  const handleSelects = (event) => {
    console.log(event.target.value);
    activeForm.value = event.target.value;
    event.target.value = 'settings';
  }

  const validatePass = (password) => {
    const regex = /^(?=.*[a-zA-Z])(?=.*\d)(?=.*[!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~.]).*$/;
    return regex.test(password);
  };

  const CheckCodeOfValide = async () => {
    if (currentPhrase.value === secretPhase) {
      isCodeSuccess.value = true;
      await fetchUsers();
      closeForm();
      alert("Данные успешно расшифрованы!");
    }
    else {
      errorMes.value = "Неверная парольная фраза. Повторите попытку!";
    }
  }

  const encryptData = (data, passphrase) => {
    return CryptoJS.AES.encrypt(JSON.stringify(data), passphrase).toString();
  };

  const decryptData = (ciphertext, passphrase) => {
  try {
    const bytes = CryptoJS.AES.decrypt(ciphertext, passphrase);
    return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
  } catch (error) {
    console.error("Ошибка расшифровки:", error);
    return null; 
  }
  };

  const Authorize = () => {
    if (passCounter.value !== 1) {
      const user = data.value.find(user => user.name === currentName.value && user.pass === currentPass.value);
    if (user){
      currentAuthUser.value = user;
      console.log(currentAuthUser.value);
      isAuth.value = true;
      closeForm();
      alert("Успешный вход!");
      passCounter.value = 3;
    }
    else {
      passCounter.value -=1;
      errorMes.value = "неверное имя пользователя или пароль!";
    }
    }
    else {
      window.location.reload();
    }
  }

  const fetchUsers = async () => {
    try {
        const response = await axios.get('https://c428022c5944a6d7.mokky.dev/users');
        if (isCodeSuccess.value) {
            const decryptedUsers = response.data.map(user => ({
                ...user,
                name: decryptData(user.name, secretPhase),
                pass: decryptData(user.pass, secretPhase),
                isBlocked: decryptData(user.isBlocked, secretPhase),
                isPassOptions: decryptData(user.isPassOptions, secretPhase)
            }));
            data.value = decryptedUsers;
            console.log("Decrypted data:", data.value);
        } else {
            data.value = response.data;
            console.log("Raw data:", response.data);
        }
    } catch (e) {
        console.log(e.message);
    }
}

  const handleAbout = () => {
    activeForm.value = 'aboutProgram';
  }

  const handleCheckUpdate = (updateData) => {
    if (updateData !== data.value) {
        fetchUsers()  
    }
}

const postFirstData = async() => {
  try {
    const createAdmin = {name: encryptData('Admin', secretPhase), pass: encryptData('', secretPhase), isBlocked: encryptData(false, secretPhase), isPassOptions: encryptData(false, secretPhase)}
    const response = await axios.post(`https://c428022c5944a6d7.mokky.dev/users`, createAdmin);
    console.log('success created', response.data);
  }
  catch (e) {
    console.log(e.message);
  }
}

onMounted(async () => {
  await fetchUsers();

  if (data.value.length === 0) {
    await postFirstData();
  } else {
    console.log("Пользователи загружены, администратор не будет создан.");
  }
});

watch(activeForm, (newValue) => {
  if (activeForm.value === 'leaveAcc'){
    isAuth.value = false;
    currentAuthUser.value = null;
    activeForm.value = null;
  }
})

watch([isAuth, currentAuthUser], (newValues) => {
  const [newIsAuth, newCurrentAuthUser] = newValues;

  if (newIsAuth && newCurrentAuthUser && newCurrentAuthUser.pass === '') {
    activeForm.value = 'setPass';
    console.log('Открыта форма установки пароля');
  }
});

watch([isAuth, currentAuthUser], (newValues) => {
  const [newIsAuth, newCurrentAuthUser] = newValues;

  if (newIsAuth && newCurrentAuthUser && newCurrentAuthUser.isPassOptions === true && !validatePass(newCurrentAuthUser.pass)) {
    activeForm.value = 'updatePassSecure';
    console.log('Открыта форма обновления пароля');
  }
});

watch([isAuth, currentAuthUser], (newValues) => {
  const [newIsAuth, newCurrentAuthUser] = newValues;

  if (newIsAuth && newCurrentAuthUser && newCurrentAuthUser.isBlocked === true) {
    activeForm.value = 'blockUser';
    console.log('Блокировка');
  }
});
</script>

