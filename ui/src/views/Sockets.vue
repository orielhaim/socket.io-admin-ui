<template>
  <div>
    <v-breadcrumbs :items="[{ title: $t('sockets.title') }]" />
    <v-card>
      <v-card-text><NamespaceSelector /></v-card-text>
      <v-data-table :headers="headers" :items="mainStore.sockets" @click:row="onRowClick">
        <template #item.transport="{ value }">
          <Transport :transport="value" />
        </template>
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
import NamespaceSelector from "../components/NamespaceSelector.vue";
import Transport from "../components/Transport.vue";
import SocketHolder from "../SocketHolder";
import { useConfigStore } from "../stores/config";
import { useMainStore } from "../stores/main";

const router = useRouter();
const configStore = useConfigStore();
const mainStore = useMainStore();

const disconnectSupported = computed(() => configStore.supportedFeatures.includes("DISCONNECT"));
const headers = computed(() => [
  { title: "#", key: "id" },
  { title: "Address", key: "handshake.address" },
  { title: "Transport", key: "transport" },
  { title: "", key: "actions", sortable: false },
]);

function disconnect(socket) {
  SocketHolder.socket?.emit("_disconnect", socket.nsp, false, socket.id);
}

function onRowClick(_event, { item }) {
  if (!mainStore.selectedNamespace) {
    return;
  }
  router.push({
    name: "socket",
    params: { nsp: mainStore.selectedNamespace.name, id: item.id },
  });
}
</script>
