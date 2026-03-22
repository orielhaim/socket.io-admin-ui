<template>
  <div>
    <v-breadcrumbs :items="breadcrumbs" />
    <v-row v-if="socket">
      <v-col cols="12" md="6">
        <v-card>
          <v-card-title>{{ $t("sockets.details") }}</v-card-title>
          <v-table>
            <tbody>
              <tr><td>ID</td><td>{{ socket.id }}</td></tr>
              <tr><td>{{ $t("sockets.address") }}</td><td>{{ socket.handshake?.address }}</td></tr>
              <tr><td>{{ $t("sockets.transport") }}</td><td><Transport :transport="socket.transport" /></td></tr>
              <tr><td>Namespace</td><td>{{ socket.nsp }}</td></tr>
            </tbody>
          </v-table>
        </v-card>
      </v-col>
      <v-col cols="12" md="6">
        <v-card>
          <v-card-title>{{ $t("rooms.title") }}</v-card-title>
          <v-list>
            <v-list-item v-for="room in socket.rooms" :key="room">
              <v-list-item-title>{{ room }}</v-list-item-title>
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

const socket = computed(() => mainStore.findSocketById(route.params.nsp, route.params.id));
const breadcrumbs = computed(() => [
  { title: "Sockets", to: { name: "sockets" } },
  { title: "Details" },
]);
</script>
