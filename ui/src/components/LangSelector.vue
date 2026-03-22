<template>
  <v-select
    v-model="lang"
    :items="languages"
    :label="$t('config.language')"
    variant="outlined"
    density="compact"
    item-title="text"
    item-value="value"
    hide-details
    class="mb-3"
  />
</template>

<script setup>
import { computed, watch } from "vue";
import { useI18n } from "vue-i18n";
import { useConfigStore } from "../stores/config";

const configStore = useConfigStore();
const i18n = useI18n();
const languages = [
  { text: "বাংলা", value: "bn" },
  { text: "English", value: "en" },
  { text: "Français", value: "fr" },
  { text: "한국어", value: "ko" },
  { text: "Português (Brazil)", value: "pt-BR" },
  { text: "Türkçe", value: "tr" },
  { text: "简体中文", value: "zh-CN" },
];

const lang = computed({
  get: () => configStore.lang,
  set: (value) => configStore.selectLang(value),
});

watch(
  () => configStore.lang,
  (value) => {
    i18n.locale.value = value;
  },
  { immediate: true },
);
</script>
