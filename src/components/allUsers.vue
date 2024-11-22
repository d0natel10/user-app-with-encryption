<template>
    <form v-if="props.activeForm === 'allUsers'" class="page__formUsers">
      <div class="page__formUsers-header">
        <h1>Все пользователи</h1>
        <img @click="props.closeForm" src="/public/icons/close.svg" alt="close window"/>
      </div>
      <div class="page__formUsers-tableContainer">
        <table class="page__formUsers-table">
          <thead>
            <tr>
              <th>#</th>
              <th>Имя пользователя</th>
              <th>Парольное ограничение</th>
              <th>Блокировка</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(user, index) in props.data" :key="user.id">
              <td>{{ index + 1 }}</td>
              <td>{{ user.name }}</td>
              <td>
                <input :disabled="user.name === 'Admin'" v-model="user.isPassOptions" type="checkbox" />
              </td>
              <td>
                <input :disabled="user.name === 'Admin'" v-model="user.isBlocked" type="checkbox" />
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="page__formUsers-formButton">
        <button @click="UpdateUsers" type="button">Сохранить</button>
      </div>
    </form>
  </template>

<script setup>
import {defineProps, defineEmits} from 'vue';
import axios from 'axios';

const props = defineProps({
    activeForm: String,
    closeForm: Function,
    data: Array,
    encryptData: Function,
    secretPhase: String
})

const emit = defineEmits();

const UpdateUsers = async () => {
    try {
      const encrypt = props.data.map(user => ({
                ...user,
                name: props.encryptData(user.name, props.secretPhase),
                pass: props.encryptData(user.pass, props.secretPhase),
                isBlocked: props.encryptData(user.isBlocked,props.secretPhase),
                isPassOptions: props.encryptData(user.isPassOptions, props.secretPhase)
        }));
        const response = await axios.patch('https://c428022c5944a6d7.mokky.dev/users', encrypt);
        alert('Данные успешно обновлены!');
        props.closeForm();
        emit('updateData', response.data);
    }
    catch (error) {
        console.log(error.message);
    }
}
</script>