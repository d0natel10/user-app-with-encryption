<template>
    <form v-if="props.activeForm === 'blockUser'" class="page__form">
      <div class="page__form-header">
        <h1>Ваш аккаунт заблокирован</h1>
      </div>
      <div class="page__form-block">
        <img src="/public/icons/blocking.png" alt="block account"/>
      </div>
    </form>
</template>

<script setup>

import { defineProps } from 'vue';

const props = defineProps({
    activeForm: String
})
</script>