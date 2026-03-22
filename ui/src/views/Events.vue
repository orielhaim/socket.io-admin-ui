<template>
  <div>
    <v-breadcrumbs :items="[{ title: $t('events.title') }]" />
    <v-card>
      <v-card-text><NamespaceSelector /></v-card-text>
      <v-data-table :headers="headers" :items="mainStore.events" item-key="eventId" :sort-by="sortBy">
        <template #item.type="{ value }"><EventType :type="value" /></template>
        <template #item.id="{ value }">
          <router-link class="plain-link" :to="socketDetailsRoute(value)">{{ value }}</router-link>
        </template>
        <template #item.args="{ item }">
          <span v-if="isExpandable(item)">{{ $t("events.eventName") }}{{ $t("separator") }}<code>{{ item.eventName }}</code></span>
          <span v-else-if="item.type === 'disconnection'">{{ $t("events.reason") }}{{ $t("separator") }}<code>{{ item.args }}</code></span>
          <span v-else-if="item.type === 'room_joined' || item.type === 'room_left'">{{ $t("events.room") }}{{ $t("separator") }}<code>{{ item.args }}</code></span>
          <span v-else>{{ item.args }}</span>
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script setup>
import NamespaceSelector from "../components/NamespaceSelector.vue";
import EventType from "../components/EventType.vue";
import { useMainStore } from "../stores/main";

const mainStore = useMainStore();
const sortBy = [
  { key: "timestamp", order: "desc" },
  { key: "eventId", order: "desc" },
];
const headers = [
  { title: "Timestamp", key: "timestamp" },
  { title: "Socket", key: "id", sortable: false },
  { title: "Type", key: "type", sortable: false },
  { title: "Data", key: "args", sortable: false },
];

function socketDetailsRoute(id) {
  return {
    name: "socket",
    params: { nsp: mainStore.selectedNamespace?.name, id },
  };
}

function isExpandable(item) {
  return item.type === "event_received" || item.type === "event_sent";
}
</script>

<style scoped>
.plain-link {
  color: inherit;
}
</style>
