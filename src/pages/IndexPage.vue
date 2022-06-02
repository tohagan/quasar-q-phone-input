<template>
  <QPhoneInput
    v-model="phone"
    v-model:valid="valid"
    dense
    outlined
    required
    label="Phone *"
    icon="phone"
    defaultCountry="AU"
    style="width: 100%; max-width: 350px; padding: 20px;"
  />

  <pre>
    phone = {{ phone }}
    valid = {{ valid }}
  </pre>
</template>

<script lang="ts" setup>
import QPhoneInput from 'src/components/phone/QPhoneInput.vue';
import { ref } from 'vue';

const valid = ref<boolean | string>(false);
const phone = ref('');
</script>
