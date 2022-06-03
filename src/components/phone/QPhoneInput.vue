<template>
  <q-input
    :tabindex="tabindex"
    :dense="dense"
    :outlined="outlined"
    :label="label"
    :maxlength="20"
    :model-value="inpNumber"
    @update:model-value="numberChanged"
    class="vue3-q-tel-input no-inherit-feedback"
    :rules="[isValidNumber]"
  >
    <template v-slot:prepend>
      <q-icon v-if="icon" :name="icon" />
      <CountrySelect
        :searchLabel="searchLabel"
        @countryChanged="countryChanged"
        v-model:country="country"
        v-bind="dropdownProps"
        class="
          q-ml-sm
          no-border-field-before no-padding-field
          font-reduced-input-adon
        "
      />
    </template>
  </q-input>
</template>

<script lang="ts" setup>
import { onMounted, ref, Ref, computed } from 'vue';
// Demo: https://catamphetamine.gitlab.io/libphonenumber-js/
import parsePhoneNumber, {
  AsYouType,
  CountryCode,
  PhoneNumber,
} from 'libphonenumber-js';

import { QInput } from 'quasar';
import CountrySelect from './CountrySelect.vue';
import { Country, getCountry } from './countries';

// import { useI18n } from 'vue-i18n';
// // eslint-disable-next-line @typescript-eslint/unbound-method
// const { t } = useI18n();

const props = defineProps({
  modelValue: { type: String, default: null, required: false }, // v-model
  valid: [Boolean, String], // v-model:valid
  required: { type: Boolean, default: false },
  searchLabel: { type: String, default: 'Search' },
  dropdownProps: { type: Object, default: () => ({}) },
  defaultCountry: { type: String, default: 'US' },
  // QInput props
  icon: String,
  label: String,
  tabindex: String,
  dense: { type: Boolean, default: false },
  outlined: { type: Boolean, default: false }
});

const emit = defineEmits<{
  (e: 'update:modelValue', intNumber?: string): void;
  // valid if === true, invalid if false or string
  (e: 'update:valid', isValid: boolean | string): void;
  (e: 'phoneChanged', phone?: PhoneNumber): void;
  (e: 'countryChanged', country?: Country): void;
  // PhoneNumber {
  //   country: 'US',
  //   countryCallingCode: '1',
  //   number: '+12133734253',
  //   nationalNumber: '2133734253',
  //   ext: '1234'
  // }
}>();

const country: Ref<Country> = ref(getCountry('US'));
const inpNumber: Ref<string> = ref('');

const defaults = computed(() => {
  const cty = country.value;
  return {
    defaultCountry: cty.iso2 as CountryCode,
    defaultCallingCode: cty.dialCode,
  };
});

onMounted(() => {
  country.value = getCountry(props.defaultCountry);
});

// onMounted(() => {
//   // modelValue is an international number, so if valid, use dial prefix to select country
//   const phone = props.modelValue && parsePhoneNumber(props.modelValue)
//   const code = phone?.country || props.defaultCountry
//   country.value = getCountry(code)
// })

function isValidNumber(val: string): boolean | string {
  if (!val) {
    return props.required ? 'This value is required' : true;
  }

  const phone = parsePhoneNumber(val, defaults.value);
  if (!phone || !phone.isValid()) {
    return 'Phone number is incomplete or incorrect';
  }

  const phType = phone.getType();
  if (
    phType &&
    !['FIXED_LINE_OR_MOBILE', 'FIXED_LINE', 'MOBILE', 'VOIP'].includes(phType)
  ) {
    return 'Personal mobile or landline number required';
  }

  return true;
}

function countryChanged() {
  numberChanged(inpNumber.value);
  emit('countryChanged', country.value);
}

function numberChanged(newNumber: string | number | null) {
  // https://github.com/catamphetamine/libphonenumber-js#as-you-type-formatter
  newNumber = newNumber?.toString() || '';
  const countryCode = country.value.iso2 as CountryCode;
  const asYouType = new AsYouType(countryCode);

  // Add leading '0' if omitted
  switch (countryCode) {
    case 'AU':
      if (/^[234578]$/.test(newNumber)) newNumber = '0' + newNumber;
    case 'NZ':
      if (/^[234679]$/.test(newNumber)) newNumber = '0' + newNumber;
  }

  // Reformat input to national formatting or international if "+" used
  inpNumber.value = newNumber = asYouType.input(newNumber);

  // Change country only if a different country dialing prefix is entered
  // Multiple contries can use the same dialCode (e.g +1 => CA,US,...)
  const newCountryCode = asYouType.getCountry() as CountryCode;
  const newCountry = newCountryCode && getCountry(newCountryCode);
  if (newCountry && country.value?.dialCode !== newCountry?.dialCode) {
    console.log(`cty=${newCountry?.iso2}`);
    country.value = newCountry; // --> countryChanged()
  }

  const valid = isValidNumber(newNumber);
  if (valid === true) {
    const newPhone = asYouType.getNumber();
    emit('update:modelValue', newPhone?.number);
    emit('phoneChanged', newPhone);
    emit('update:valid', true);
  } else {
    emit('update:modelValue', undefined);
    emit('phoneChanged', undefined);
    emit('update:valid', valid);
  }
}
</script>
