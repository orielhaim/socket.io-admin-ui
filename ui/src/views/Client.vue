<template>
  <div>
    <v-breadcrumbs :items="breadcrumbs" />
    <v-row v-if="client">
      <v-col cols="12" md="4">
        <v-card>
          <v-card-title>{{ $t("clients.details") }}</v-card-title>
          <v-table>
            <tbody>
              <tr><td>ID</td><td>{{ client.id }}</td></tr>
              <tr><td>{{ $t("clients.sockets-count") }}</td><td>{{ client.sockets.length }}</td></tr>
            </tbody>
          </v-table>
        </v-card>
      </v-col>
      <v-col cols="12" md="8">
        <v-card>
          <v-card-title>{{ $t("sockets.title") }}</v-card-title>
          <v-list>
            <v-list-item
              v-for="socket in client.sockets"
              :key="socket.id"
              :to="{ name: 'socket', params: { nsp: socket.nsp, id: socket.id } }"
            >
              <v-list-item-title>{{ socket.id }}</v-list-item-title>
              <template #append><Transport :transport="socket.transport" /></template>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { useRoute } from "vue-router";
import Transport from "../components/Transport.vue";
import { useMainStore } from "../stores/main";

const route = useRoute();
const mainStore = useMainStore();
const client = computed(() => mainStore.findClientById(route.params.id));
const breadcrumbs = computed(() => [
  { title: "Clients", to: { name: "clients" } },
  { title: "Details" },
]);
</script>
