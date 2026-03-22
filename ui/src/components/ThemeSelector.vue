<template>
  <v-switch
    v-model="darkTheme"
    :label="$t('config.dark-theme')"
    density="compact"
    hide-details
    inset
    class="mb-3"
  />
</template>

<script setup>
import { computed } from "vue";
import { useConfigStore } from "../stores/config";

const configStore = useConfigStore();
const darkTheme = computed({
  get: () => configStore.darkTheme,
  set: (value) => configStore.selectTheme(value),
});
</script>
