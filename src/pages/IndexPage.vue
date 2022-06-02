quas<template>
  <q-page>
    <PhoneNum
      v-model="phone"
      v-model:valid="isPhoneValid"
      dense
      outlined
      required
      label="Phone *"
      icon="phone"
      defaultCountry="AU"
      style="width: 100%; max-width: 350px"
    />

    <pre>
      phone = {{ phone }}
      isPhoneValid = {{ isPhoneValid }}
    </pre>
  </q-page>
</template>

<script lang="ts" setup>
import PhoneNum from 'src/components/phone/PhoneNum.vue';
import { ref } from 'vue';

const isPhoneValid = ref<boolean | undefined>(false);
const phone = ref(string);
</script>
