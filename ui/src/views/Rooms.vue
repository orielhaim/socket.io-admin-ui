<template>
  <div>
    <v-breadcrumbs :items="[{ title: $t('rooms.title') }]" />
    <v-card>
      <v-card-text class="d-flex align-center ga-4">
        <NamespaceSelector />
        <v-switch v-model="showPrivateRooms" :label="$t('rooms.show-private')" hide-details inset />
      </v-card-text>
      <v-data-table :headers="headers" :items="filteredRooms" @click:row="onRowClick">
        <template #item.isPrivate="{ value }"><RoomType :is-private="value" /></template>
        <template #item.sockets="{ item }">{{ item.sockets.length }}</template>
        <template #item.actions="{ item }">
          <v-btn
            v-if="multiLeaveSupported && !item.isPrivate"
            icon="mdi-tag-off-outline"
            size="small"
            variant="text"
            :disabled="configStore.readonly"
            @click.stop="clear(item)"
          />
          <v-btn
            v-if="multiDisconnectSupported"
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
import { computed, ref, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import { sortBy } from "lodash-es";
import NamespaceSelector from "../components/NamespaceSelector.vue";
import RoomType from "../components/Room/RoomType.vue";
import SocketHolder from "../SocketHolder";
import { useConfigStore } from "../stores/config";
import { useMainStore } from "../stores/main";

const route = useRoute();
const router = useRouter();
const configStore = useConfigStore();
const mainStore = useMainStore();
const showPrivateRooms = ref(route.query.p === "1");

const multiLeaveSupported = computed(() => configStore.supportedFeatures.includes("MLEAVE"));
const multiDisconnectSupported = computed(() => configStore.supportedFeatures.includes("MDISCONNECT"));
const headers = [
  { title: "ID", key: "name" },
  { title: "Type", key: "isPrivate" },
  { title: "Sockets", key: "sockets" },
  { title: "", key: "actions", sortable: false },
];

const filteredRooms = computed(() => {
  const rooms = showPrivateRooms.value
    ? mainStore.rooms
    : mainStore.rooms.filter((room) => !room.isPrivate);
  return sortBy(rooms, "name");
});

watch(showPrivateRooms, (value) => {
  router.replace({ name: "rooms", query: value ? { p: "1" } : {} });
});

function clear(room) {
  SocketHolder.socket?.emit("leave", mainStore.selectedNamespace?.name, room.name);
}

function disconnect(room) {
  SocketHolder.socket?.emit(
    "_disconnect",
    mainStore.selectedNamespace?.name,
    false,
    room.name,
  );
}

function onRowClick(_event, { item }) {
  if (!mainStore.selectedNamespace) {
    return;
  }
  router.push({
    name: "room",
    params: { nsp: mainStore.selectedNamespace.name, name: item.name },
  });
}
</script>
