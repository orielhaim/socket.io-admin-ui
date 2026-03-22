<template>
  <div>
    <v-breadcrumbs :items="[{ title: $t('clients.title') }]" />
    <v-card>
      <v-data-table :headers="headers" :items="mainStore.clients" @click:row="onRowClick">
        <template #item.address="{ item }">{{ item.sockets[0]?.handshake?.address || "-" }}</template>
        <template #item.transport="{ item }"><Transport :transport="item.sockets[0]?.transport" /></template>
        <template #item.sockets="{ item }">{{ item.sockets.length }}</template>
        <template #item.actions="{ item }">
          <v-btn
            v-if="disconnectSupported"
            icon="mdi-logout"
            size="small"
            variant="text"
            :disabled="configStore.readonly"
            @click.stop="disconnect(item)"
          />
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { useRouter } from "vue-router";
import Transport from "../components/Transport.vue";
import SocketHolder from "../SocketHolder";
import { useConfigStore } from "../stores/config";
import { useMainStore } from "../stores/main";

const router = useRouter();
const configStore = useConfigStore();
const mainStore = useMainStore();
const disconnectSupported = computed(() => configStore.supportedFeatures.includes("DISCONNECT"));

const headers = [
  { title: "#", key: "id" },
  { title: "Address", key: "address" },
  { title: "Transport", key: "transport" },
  { title: "Sockets", key: "sockets" },
  { title: "", key: "actions", sortable: false },
];

function disconnect(client) {
  const socket = client.sockets[0];
  if (socket) {
    SocketHolder.socket?.emit("_disconnect", socket.nsp, true, socket.id);
  }
}

function onRowClick(_event, { item }) {
  router.push({ name: "client", params: { id: item.id } });
}
</script>
