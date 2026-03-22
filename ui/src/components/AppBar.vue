<template>
  <v-app-bar>
    <v-app-bar-nav-icon class="d-lg-none" @click="configStore.toggleNavigationDrawer()" />
    <v-img :src="logoSrc" alt="logo" max-height="40" max-width="40" />
    <v-toolbar-title class="ml-3">Socket.IO Admin UI</v-toolbar-title>
    <v-btn variant="text" size="small" class="ml-2 text-none" :href="releaseNotesLink" target="_blank">
      v{{ version }}
    </v-btn>

    <v-spacer />

    <div class="d-none d-lg-flex align-center ga-3">
      <div class="text-caption">
        <div>{{ $t("connection.serverUrl") }}{{ $t("separator") }}<code>{{ connectionStore.serverUrl || "-" }}</code></div>
        <div>{{ $t("status") }}{{ $t("separator") }}<ConnectionStatus :connected="connectionStore.connected" /></div>
      </div>
      <v-btn variant="outlined" @click="$emit('update')">{{ $t("update") }}</v-btn>
    </div>
  </v-app-bar>
</template>

<script setup>
import { computed } from "vue";
import ConnectionStatus from "./ConnectionStatus.vue";
import { useConfigStore } from "../stores/config";
import { useConnectionStore } from "../stores/connection";
import darkLogo from "../assets/logo-dark.svg";
import lightLogo from "../assets/logo-light.svg";

const configStore = useConfigStore();
const connectionStore = useConnectionStore();

const version = __APP_VERSION__;
const releaseNotesLink = `https://github.com/socketio/socket.io-admin-ui/releases/tag/${version}`;
const logoSrc = computed(() => (configStore.darkTheme ? darkLogo : lightLogo));
</script>
