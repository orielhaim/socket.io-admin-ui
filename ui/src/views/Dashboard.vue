<template>
  <div>
    <v-breadcrumbs :items="[{ title: $t('dashboard.title') }]" />
    <v-row>
      <v-col cols="12" md="4">
        <v-card>
          <v-card-title class="d-flex align-center justify-space-between">
            <span>{{ $t("clients.title") }}</span>
            <v-menu>
              <template #activator="{ props }">
                <v-btn icon="mdi-dots-vertical" size="small" variant="text" v-bind="props" />
              </template>
              <v-list density="compact">
                <v-list-item title="Overview" />
              </v-list>
            </v-menu>
          </v-card-title>
          <v-card-text>
            <div class="text-h5 mb-3">{{ mainStore.clients.length }}</div>
            <div class="d-flex flex-wrap ga-2">
              <v-chip color="success" variant="tonal" size="small">
                WebSocket: {{ transportBreakdown.websocket }}
              </v-chip>
              <v-chip color="warning" variant="tonal" size="small">
                HTTP long-polling: {{ transportBreakdown.polling }}
              </v-chip>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card>
          <v-card-title class="d-flex align-center justify-space-between">
            <span>{{ $t("servers.title") }}</span>
            <v-menu>
              <template #activator="{ props }">
                <v-btn icon="mdi-dots-vertical" size="small" variant="text" v-bind="props" />
              </template>
              <v-list density="compact">
                <v-list-item title="Overview" />
              </v-list>
            </v-menu>
          </v-card-title>
          <v-card-text>
            <div class="chart-box">
              <Doughnut :data="serversDonutData" :options="donutOptions" />
            </div>
            <div class="d-flex flex-wrap ga-2 mt-2">
              <v-chip color="success" variant="tonal" size="small">
                Healthy: {{ healthyServers }} ({{ healthyPercent }}%)
              </v-chip>
              <v-chip color="error" variant="tonal" size="small">
                Unhealthy: {{ unhealthyServers }} ({{ unhealthyPercent }}%)
              </v-chip>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card>
          <v-card-title class="d-flex align-center justify-space-between">
            <span>{{ $t("select-namespace") }}</span>
            <v-menu>
              <template #activator="{ props }">
                <v-btn icon="mdi-dots-vertical" size="small" variant="text" v-bind="props" />
              </template>
              <v-list density="compact">
                <v-list-item title="Overview" />
              </v-list>
            </v-menu>
          </v-card-title>
          <v-card-text class="pt-0">
            <v-table density="compact">
              <thead>
                <tr>
                  <th>Namespace</th>
                  <th class="text-right">Sockets</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in namespaceRows" :key="item.name">
                  <td>{{ item.name }}</td>
                  <td class="text-right">{{ item.socketsCount }}</td>
                </tr>
                <tr v-if="namespaceRows.length === 0">
                  <td colspan="2" class="text-medium-emphasis">-</td>
                </tr>
              </tbody>
            </v-table>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col v-if="configStore.hasAggregatedValues" cols="12" md="6">
        <v-card>
          <v-card-title class="d-flex align-center justify-space-between">
            <span>Connection and disconnection events</span>
            <v-menu>
              <template #activator="{ props }">
                <v-btn icon="mdi-dots-vertical" size="small" variant="text" v-bind="props" />
              </template>
              <v-list density="compact">
                <v-list-item title="Overview" />
              </v-list>
            </v-menu>
          </v-card-title>
          <v-card-text>
            <div class="chart-box">
              <Bar :data="connectionsData" :options="chartOptions" />
            </div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col v-if="configStore.hasAggregatedValues" cols="12" md="6">
        <v-card>
          <v-card-title class="d-flex align-center justify-space-between">
            <span>{{ bytesChartTitle }}</span>
            <v-menu>
              <template #activator="{ props }">
                <v-btn icon="mdi-dots-vertical" size="small" variant="text" v-bind="props" />
              </template>
              <v-list density="compact">
                <v-list-item title="Overview" />
              </v-list>
            </v-menu>
          </v-card-title>
          <v-card-text>
            <div class="chart-box">
              <Bar :data="bytesData" :options="chartOptions" />
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { Bar, Doughnut } from "vue-chartjs";
import { sortBy } from "lodash-es";
import { subMinutes } from "date-fns";
import { useConfigStore } from "../stores/config";
import { useMainStore } from "../stores/main";
import { useServersStore } from "../stores/servers";

const configStore = useConfigStore();
const mainStore = useMainStore();
const serversStore = useServersStore();
const now = new Date();

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  parsing: false,
  animation: false,
  plugins: {
    legend: {
      position: "top",
      labels: {
        usePointStyle: true,
        boxWidth: 10,
      },
    },
    tooltip: {
      intersect: false,
      mode: "index",
    },
  },
  scales: {
    x: {
      type: "time",
      min: subMinutes(now, 10),
      max: now,
      time: {
        unit: "minute",
        stepSize: 1,
      },
      grid: {
        color: "rgba(127,127,127,0.15)",
      },
    },
    y: {
      beginAtZero: true,
      ticks: {
        precision: 0,
      },
      grid: {
        color: "rgba(127,127,127,0.15)",
      },
    },
  },
};

const donutOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: "bottom",
    },
  },
};

const transportBreakdown = computed(() => {
  const totals = {
    websocket: 0,
    polling: 0,
  };
  for (const client of mainStore.clients) {
    const transport = client.sockets[0]?.transport;
    if (transport === "websocket") {
      totals.websocket += 1;
    } else if (transport === "polling") {
      totals.polling += 1;
    }
  }
  return totals;
});

const healthyServers = computed(
  () => serversStore.servers.filter((server) => server.healthy).length,
);
const unhealthyServers = computed(
  () => serversStore.servers.filter((server) => !server.healthy).length,
);
const serversTotal = computed(() => healthyServers.value + unhealthyServers.value);
const healthyPercent = computed(() =>
  serversTotal.value ? ((healthyServers.value / serversTotal.value) * 100).toFixed(1) : "0.0",
);
const unhealthyPercent = computed(() =>
  serversTotal.value ? ((unhealthyServers.value / serversTotal.value) * 100).toFixed(1) : "0.0",
);
const serversDonutData = computed(() => ({
  labels: ["Healthy", "Unhealthy"],
  datasets: [
    {
      data: [healthyServers.value, unhealthyServers.value],
      backgroundColor: ["rgba(76, 175, 80, 0.8)", "rgba(244, 67, 54, 0.8)"],
      borderWidth: 0,
      cutout: "60%",
    },
  ],
}));

const namespaceRows = computed(() => {
  if (serversStore.namespaces.length) {
    return sortBy(serversStore.namespaces, "name");
  }
  return sortBy(
    mainStore.namespaces.map((namespace) => ({
      name: namespace.name,
      socketsCount: namespace.sockets.length,
    })),
    "name",
  );
});

const aggregatedEvents = computed(() => mainStore.aggregatedEvents || []);

function mapAggregatedEvent(event) {
  return {
    x: event.timestamp,
    y: event.count,
  };
}

const connectionEvents = computed(() =>
  aggregatedEvents.value
    .filter((event) =>
      ["rawConnection", "connections", "connection"].includes(event.type),
    )
    .map(mapAggregatedEvent),
);

const disconnectionEvents = computed(() =>
  aggregatedEvents.value
    .filter((event) =>
      ["rawDisconnection", "disconnections", "disconnection"].includes(event.type),
    )
    .map(mapAggregatedEvent),
);

const bytesInSeries = computed(() =>
  aggregatedEvents.value
    .filter((event) => {
      if (event.type === "bytesIn") return true;
      if (event.type === "bytes") return ["received", "in"].includes(event.subType);
      if (event.type === "packets") return ["received", "in"].includes(event.subType);
      return false;
    })
    .map(mapAggregatedEvent),
);

const bytesOutSeries = computed(() =>
  aggregatedEvents.value
    .filter((event) => {
      if (event.type === "bytesOut") return true;
      if (event.type === "bytes") return ["sent", "out"].includes(event.subType);
      if (event.type === "packets") return ["sent", "out"].includes(event.subType);
      return false;
    })
    .map(mapAggregatedEvent),
);

const hasNativeBytesSeries = computed(() =>
  aggregatedEvents.value.some((event) => event.type === "bytesIn" || event.type === "bytesOut"),
);

const bytesChartTitle = computed(() =>
  hasNativeBytesSeries.value ? "Bytes received and sent" : "Packets received and sent",
);

const connectionsData = computed(() => ({
  datasets: [
    {
      label: "Connections",
      data: connectionEvents.value,
      backgroundColor: "rgba(76, 175, 80, 0.72)",
      borderColor: "rgba(76, 175, 80, 1)",
      borderRadius: 6,
      barThickness: 8,
    },
    {
      label: "Disconnections",
      data: disconnectionEvents.value,
      backgroundColor: "rgba(244, 67, 54, 0.72)",
      borderColor: "rgba(244, 67, 54, 1)",
      borderRadius: 6,
      barThickness: 8,
    },
  ],
}));

const bytesData = computed(() => ({
  datasets: [
    {
      label: hasNativeBytesSeries.value ? "Bytes received" : "Packets received",
      data: bytesInSeries.value,
      borderColor: "rgba(33, 150, 243, 1)",
      backgroundColor: "rgba(33, 150, 243, 0.72)",
      borderRadius: 6,
      barThickness: 8,
    },
    {
      label: hasNativeBytesSeries.value ? "Bytes sent" : "Packets sent",
      data: bytesOutSeries.value,
      borderColor: "rgba(239, 83, 80, 1)",
      backgroundColor: "rgba(239, 83, 80, 0.72)",
      borderRadius: 6,
      barThickness: 8,
    },
  ],
}));
</script>

<style scoped>
.chart-box {
  height: 220px;
}
</style>
