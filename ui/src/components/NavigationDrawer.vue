<template>
  <v-navigation-drawer v-model="drawerOpen" elevation="3">
    <v-list nav>
      <v-list-item
        v-for="item in items"
        :key="item.title"
        :to="item.to"
        :exact="item.exact"
        :prepend-icon="item.icon"
      >
        <v-list-item-title>{{ item.title }}</v-list-item-title>
      </v-list-item>
    </v-list>

    <template #append>
      <v-divider />
      <div class="pa-4 pt-8">
        <LangSelector />
        <ThemeSelector />
        <ReadonlyToggle />
      </div>
    </template>
  </v-navigation-drawer>
</template>

<script setup>
import { computed } from "vue";
import { useI18n } from "vue-i18n";
import LangSelector from "./LangSelector.vue";
import ThemeSelector from "./ThemeSelector.vue";
import ReadonlyToggle from "./ReadonlyToggle.vue";
import { useConfigStore } from "../stores/config";

const { t } = useI18n();
const configStore = useConfigStore();

const drawerOpen = computed({
  get() {
    return configStore.showNavigationDrawer;
  },
  set(value) {
    configStore.showNavigationDrawer = value;
  },
});

const items = computed(() => {
  if (!configStore.developmentMode) {
    return [
      { title: t("dashboard.title"), icon: "mdi-home-outline", to: { name: "dashboard" }, exact: true },
      { title: t("servers.title"), icon: "mdi-server", to: { name: "servers" } },
    ];
  }
  return [
    { title: t("dashboard.title"), icon: "mdi-home-outline", to: { name: "dashboard" }, exact: true },
    { title: t("sockets.title"), icon: "mdi-ray-start-arrow", to: { name: "sockets" } },
    { title: t("rooms.title"), icon: "mdi-tag-outline", to: { name: "rooms" } },
    { title: t("clients.title"), icon: "mdi-account-circle-outline", to: { name: "clients" } },
    { title: t("events.title"), icon: "mdi-calendar-text-outline", to: { name: "events" } },
    { title: t("servers.title"), icon: "mdi-server", to: { name: "servers" } },
  ];
});
</script>
