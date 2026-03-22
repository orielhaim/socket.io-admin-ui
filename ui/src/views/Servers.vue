<template>
  <div>
    <v-breadcrumbs :items="[{ title: $t('servers.title') }]" />
    <v-card>
      <v-data-table :headers="headers" :items="sortedServers">
        <template #item.uptime="{ value }">{{ formatDuration(value) }}</template>
        <template #item.lastPing="{ value }">{{ delaySinceLastPing(value) }}</template>
        <template #item.healthy="{ value }"><ServerStatus :healthy="value" /></template>
        <template #item.actions="{ item }">
          <v-btn v-if="!item.healthy" icon="mdi-delete-outline" size="small" variant="text" @click="serversStore.removeServer(item.serverId)" />
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue";
import { sortBy } from "lodash-es";
import { formatDuration } from "../util";
import ServerStatus from "../components/ServerStatus.vue";
import { useServersStore } from "../stores/servers";

const serversStore = useServersStore();
const now = ref(Date.now());
let timer = null;

const headers = [
  { title: "ID", key: "serverId" },
  { title: "Hostname", key: "hostname" },
  { title: "PID", key: "pid" },
  { title: "Uptime", key: "uptime" },
  { title: "Clients", key: "clientsCount" },
  { title: "Last ping", key: "lastPing" },
  { title: "Status", key: "healthy" },
  { title: "", key: "actions", sortable: false },
];

const sortedServers = computed(() => sortBy(serversStore.servers, "serverId"));

function delaySinceLastPing(lastPing) {
  return `${formatDuration((now.value - lastPing) / 1000)} ago`;
}

onMounted(() => {
  timer = setInterval(() => {
    now.value = Date.now();
  }, 1000);
});

onUnmounted(() => {
  clearInterval(timer);
});
</script>
