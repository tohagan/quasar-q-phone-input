<template>
  <q-select
    ref="selectCountry"
    hide-bottom-space
    hide-dropdown-icon
    borderless
    dense
    standout="bg-grey-3"
    :model-value="country"
    @update:model-value="countryChanged"
    @popup-show="onShow"
    :options="countryOptions"
    virtual-scroll-slice-size="9999"
    class="no-inherit-feedback no-feedback v3-q-tel-input--country"
    :menu-offset="[12, 0]"
    v-bind="props"
    style="width: 70px"
  >
    <template v-slot:option="scope">
      <div
        class="flex items-center q-pa-xs mdi-border-bottom no-wrap"
        v-bind="scope.itemProps"
      >
        <span :class="['v3q_tel__flag', scope.opt.iso2.toLowerCase()]"></span>
        <span class="q-ml-sm text-no-wrap">(+{{ scope.opt.dialCode }})</span>
        <span class="q-ml-sm text-no-wrap ellipsis">{{ scope.opt.name }}</span>
      </div>
      <q-separator />
    </template>
    <template v-slot:selected-item="scope">
      <div
        v-if="scope.opt"
        class="q-pa-none ellipsis"
        style="min-height: unset"
      >
        <div class="flex items-center no-wrap">
          <span
            :class="['v3q_tel__flag q-mr-sm', scope.opt.iso2.toLowerCase()]"
          ></span>
          <span
            class="ellipsis text-no-wrap"
            v-html="`+${scope.opt.dialCode}`"
          ></span>
        </div>
      </div>
    </template>
    <template v-slot:no-option>
      <!-- Same as v-slot:before-options -->
      <q-input
        class="sticky-input z-top v3-q-tel--country-selector q-pa-sm bg-white"
        ref="search"
        :label="searchLabel"
        v-model="searchText"
        @update:model-value="performSearch"
        dense
        outlined
      >
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>

      <div class="q-pa-sm">None match</div>
    </template>
    <template v-slot:before-options>
      <!-- Same as v-slot:before-no-option -->
      <q-input
        class="sticky-input z-top v3-q-tel--country-selector q-pa-sm bg-white"
        ref="search"
        :label="searchLabel"
        v-model="searchText"
        @update:model-value="performSearch"
        dense
        outlined
      >
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>
    </template>
  </q-select>
</template>

<style lang="scss">
@import 'src/css/flags.css';
// .q-field--outlined .q-field__control:hover:before {
//   border-color: #375f7d;
//   border-width: 3px;
// }
.sticky-input {
  position: -webkit-sticky; /* Safari */
  position: sticky;
  top: 0;
}

.q-field--outlined .v3-q-tel-input--country {
  .q-field__control:after {
    border: none !important;
  }
}

.v3-q-tel-input--country {
  .q-field__control {
    .q-field__native {
      padding-top: 2px;
    }

    background: none !important;
    &::before {
      display: none !important;
    }
  }
  .q-field__input {
    outline: none !important;
    border: none !important;
  }
}

.v3-q-tel--country-selector {
  position: sticky !important;
  bottom: 0 !important;
  left: 0 !important;
  right: 0 !important;
}
</style>

<script lang="ts" setup>
// Adapted from ...
// - https://github.com/iamstevendao/vue-tel-input/blob/master/LICENSE
// Copyright (c) 2018 Steven Dao - MIT License

import { ref, onMounted, nextTick } from 'vue';
import { filterCountries, Country, countries } from './countries';
import { QSelect, QInput } from 'quasar';
// import { delay } from 'src/utils';

const props = defineProps<{
  country: Country;
  searchLabel: string;
}>();

const emit = defineEmits(['countryChanged', 'update:country']);

const selectCountry = ref<QSelect>();
const search = ref<QInput>();
const searchText = ref('');
const countryOptions = ref([...countries]);

function performSearch() {
  const needle = searchText.value.toLowerCase().trim();
  countryOptions.value =
    needle === '' ? [...countries] : filterCountries(needle);
}

let prevCountry: Country;
onMounted(() => {
  prevCountry = props.country;
});

async function onShow() {
  console.log('onShow');
  searchText.value = '';
  performSearch();
  // render search QInput & QSelect before focus
  await nextTick();
  search.value?.focus();
}

function countryChanged(val: Country) {
  emit('update:country', val);
  emit('countryChanged', val, prevCountry);
  prevCountry = val;
}
</script>
