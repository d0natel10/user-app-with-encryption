<template>
    <form v-if="props.activeForm === 'aboutProgram'" class="page__form">
      <div class="page__form-header">
        <h1>О программе</h1>
        <img @click="close" src="/public/icons/close.svg" alt="close window"/>
      </div>
      <div class="page__form-info">
        <p>Работу выполнил студент группы ИДБ-21-01</p>
        <p>Лямцев Кирилл</p>
        <p>Индивидуальное задание - Наличие букв, цифр и знаков препинания.</p>
      </div>
    </form>
</template>

<script setup> 
import  { defineProps } from 'vue';

const props = defineProps({
    activeForm: String,
    closeForm: Function,
})

const close = () => {
    props.closeForm();
}
</script>