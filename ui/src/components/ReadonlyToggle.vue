<template>
  <v-switch
    v-model="readonly"
    :label="$t('config.readonly')"
    :disabled="disabled"
    density="compact"
    hide-details
    inset
  />
</template>

<script setup>
import { computed } from "vue";
import { useConfigStore } from "../stores/config";

const configStore = useConfigStore();
const disabled = computed(() => configStore.supportedFeatures.length === 0);
const readonly = computed({
  get: () => configStore.readonly,
  set: () => configStore.toggleReadonly(),
});
</script>
